# Save/restore state design proposal

This document provides a high level idea about how we can save and restore the
state of a component in rust-vmm such that it is compatible with (at least) the
known VMMs (Firecracker, cloud-hypervisor), and to get the community’s input on
the proposal. We need to add this support in rust-vmm to enable snapshot use
cases, such as live migration, and to unblock the consumption of components
(e.g. virtio devices, legacy devices).

A rust-vmm component (and any component for that matter), will have some well
defined fields (e.g. `u64`, `String`, `Vec<u8>`), and/or some generic ones (e.g.
`T: Trigger`, `W: std::io::Write`). We call this component `Abc` throughout the
document. Example:

```
pub struct Abc<EV: AbcEvents, W: std::io::Write, ...> {
    // Well defined fields.
    a: u32,
    b: String,
    ...
    // Generic fields.
    events: EV,
    out: W,
    ...
}
```

The state of this component will be stored in a similar structure, `AbcState`:

```
pub struct AbcState {
    // Well defined fields.
    pub a: u32,
    pub b: String,
    ...
    // End of well defined fields.
}
```

### Simplifying assumptions

We propose this structure to hold, at least for now, the state of only the well
defined fields, and not the one of the generic ones due to complexity reasons.
There is a [section](#State-content) in this document about this subject. For
most use cases we expect the state of the generic components will not be needed
after restore. If we later notice that for a particular device, it would make
more sense to save the state of the generics as well, then we can adapt the
current design to fit this use case.

## Tenets

When implementing/maintaining the state of components, we’ll always be guided
by the following:

1. The state does not include implementation details. For devices, the state is
   1:1 with the specification.
2. By default, we offer support for upgrading/downgrading to any version. Any
   deviations to this rule are documented, and discussed beforehand with the
   rust-vmm customers.
3. The translations between different versions are tested (including
   performance testing with micro-benchmarks).
4. The state is separated from any (de)serialization or versioning logic.

## Requirements

When designing the save/restore state approach, we keep in mind the following
functional and non-functional requirements:

### Functional

* The same abstraction (or approach) can be used for all components defined in
  rust-vmm.
* The state abstractions can be seamlessly integrated in customers’ products,
  without losing any backwards compatibility. These abstractions should
  therefore be compatible or provide a mechanism for reaching compatibility with
  `Versionize` trait (and also `serde`'s `Serialize` and `Deserialize`).

### Non-functional

* We avoid having different code paths that achieve the same purpose in the
  devices' implementation (by always creating the component from a state).
* We keep the extra dependencies on base crates to a minimum (for example 
  `vm-superio` has 0 dependencies). In practice, this is achieved through
  separating the state from the serialization logic (as per Tenet 4).

We make a distinction between state that can be used to save/restore objects,
and state that can be persisted (e.g. de(serialized), versionized) as well. Our
goal is to support the persistent state as an optional feature such that
customers that do not need it, are not forced to use it. One potential problem
we are seeing is that different customers of the same logical rust-vmm component
(for example virtio-queue) need to persist different fields of the same
component. We propose saving all the fields that correspond to the state in
rust-vmm. To account for incompatibilities between the rust-vmm state and the
already existing consumers state, we propose implementing `From` (or similar
mechanisms) in the consumer products to convert the upstream state to the
desired product state. We will work with costumers as we add save/restore to
existing components to make sure that such translations are possible.

## Proposal

Each component `Abc` has an `AbcState` defined in the same module and crate with
the base component. To avoid branching in the calling functions, the `Abc`
component will always be created from a state (even if it’s an empty one).
In order to not break the existing interface, `new()` and `with_events()`
(which are receiving as parameters only the generic objects, e.g.
`Serial::new(trigger: T, out: W)`, `Serial::with_events(trigger: T,
serial_evts: EV, out: W)`) have the same interface as before, and are calling
`from_state()` with the `Default` implementation of the state object. We expect
this assumption holds for each state in rust-vmm which doesn’t keep the state
of generics. If we’d save the state of the generics as well, then the interface
of the component would suffer modifications.

**Example** (the `Abc` component has also a generic `AbcEvents` in its
structure, similar to the `RTC` device for example):

```
impl Abc<NoEvents> {
    /// Creates a new `Abc` instance without any metric capabilities and from a
    /// default state.
    pub fn new() -> Self {
        Self::from_state(AbcState::default(), NoEvents)
    }
}

impl<EV: AbcEvents> Abc<EV> {
    /// Creates a new `Abc` instance from a given `state` and invokes the
    /// `events` implementation of `AbcEvents` during operation.
    ///
    /// # Arguments
    /// * `state` - The state from which the Abc component is constructed.
    /// * `events` - The `AbcEvents` implementation used to track the occurrence
    ///              of the events in the `Abc` operation.
    pub fn from_state(state: AbcState, events: EV) -> Self {
        Abc {
            a: state.a,
            ...
            events
        }
    }

    /// Creates a new `Abc` instance from the default state and invokes the
    /// `events` implementation of `AbcEvents` during operation.
    ///
    /// # Arguments
    /// * `events` - The `AbcEvents` implementation used to track the occurrence
    ///              of the events in the `Abc` operation.
    pub fn with_events(events: EV) -> Self {
        Self::from_state(AbcState::default(), events)
    }
}
```

For getting the state of the component, there will be a getter for it in the
`Abc` interface.

```
/// `Abc` State.
pub struct AbcState {
    // These are `pub`s so they can be accessed by customers, when not
    // the entire `AbcState` is relevant.
    pub a: u32,
    pub b: u64,
...
}

impl<EV: AbcEvents> Abc<EV> {
    /// Returns the state of the `Abc` component.
    pub fn state(&self) -> AbcState {
        AbcState {
            a: self.a,
            ...
        }
    }
}
```

These state structures won’t derive/implement any serialization trait. For the
(de)ser and versioning support, we convert the crate containing the component
(named, for example, `component`) to a workspace, in which we have the following
crates:

- `component` - which keeps the state of that component as well;
- `component-ser` - which mirrors the state structure from component and adds
  the required version constraints on it, and derives/implements the required
  (de)serialization traits (i.e. `Serialize`, `Deserialize`, `Versionize`).

Below you can see an example of such state wrapper. In case for example we
decide to add a new field in the base component (i.e. `component` crate),
`AbcStateSer` will have to be updated as well (and if it’s required, to also
add a `#[version(start = 2)]` to it for the `Versionize` use case).

```
/// Wrapper over an `AbcState` that has serialization capabilities.
#[derive(Clone, Debug, PartialEq, Deserialize, Serialize, Versionize)]
pub struct AbcStateSer {
    pub a: u32,
    pub b: u64,
    ...
    #[version(start = 2)]
    pub some_new_field: Vec<u8>
}
```

For the consumer to be able to obtain an `Abc` component from an `AbcStateSer`
(needed on the restore path), we will provide a `From` implementation for that
in `component-ser`.

```
impl From<&AbcStateSer> for AbcState {
    fn from(state: &AbcStateSer) -> Self {
        AbcState {
            a: state.a,
            b: state.b,
            ...
            some_new_field: state.some_new_field.clone()
        }
    }
}
```

And on the save state path, the `AbcStateSer` interface can be extended as
follows:

```
impl AbcStateSer {
    /// Creates a new AbcStateSer from an AbcState object.
    pub fn new(state: AbcState) -> AbcStateSer {
        AbcStateSer {
            a: state.a,
            b: state.b,
            ...
            new_field: state.new_field,
        }
    }
}
```

This proposal comes with the following disadvantages:

* the serialized/versionized state can be consumed directly from upstream
  *only if* that’s the exact state that’s needed. If the customer needs a
  different state, then a `From` mechanism will have to be implemented by them.
  We expect this to not be needed when there’s a new device in rust-vmm that was
  not yet defined in VMMs and will be consumed from upstream from the beginning.
* two customers can’t have different states of the same component at the same
  version; they must be in sync regarding the content of `AbcStateSer` at each
  version.
* requires conversion mechanisms in the VMM integrating the component for
  snapshot support *if* not the entire state from upstream is needed.

### Handling incompatibilities between states

If the state from `component-ser` is not the exact state that is needed by the
consumer, then the user can either implement the `From` trait or define custom
functions on the states from `component` (e.g. `from_state()`) in the VMM, as
it can be seen in the following example:

```
#[derive(Deserialize, Serialize, Versionize)]
pub struct OtherVersionizedAbcState {
    pub other_state: OtherRelevantVersionizedState,
    pub a: u32,
    pub b: i64,
}

// Can be used for converting from the base state to a versionized one
// (another option here would be to add a `From` implementation from
// `(other_relevant_state, abc_state)` tuple).
impl OtherVersionizedAbcState {
    pub fn from_state(other_relevant_state: &OtherState, abc_state: &AbcState)
     -> Self {
        Self {
            other_state: OtherRelevantVersionizedState::new(
                other_relevant_state.x, ...),
            a: abc_state.a,
            b: abc_state.b,
        }
    }
}

// Can be used for converting from a versionized state to the base one.
impl From<&OtherVersionizedAbcState> for AbcState {
    fn from(abc_state: &OtherVersionizedAbcState) -> Self {
        Self {
            a: abc_state.a,
            b: abc_state.b,
            c: 0,
            ...
        }
    }
}
```

## State content

We propose to keep in the upstream state only the well defined fields for now.
The generics add a lot of complexity and restrictions to the consumers, for
example:

* The `Clone` trait bound will be required for those generics. Let’s take the
  events example. If the `AbcEvents` would be kept in the state as well, we will
  have to add the `Clone` trait bound to the `AbcEvents` (that is needed by the
  getter of the state). This will require the `AbcEvents` concrete
  implementations in the VMM products  to implement (or just derive when
  possible) the `Clone` trait. Even when the `AbcEvents` objects are keeping
  just Atomics, `Clone` will have to be manually implemented. There might also
  be cases when `Clone` is difficult or undesirable to implement.
* The breaking changes can’t be kept to a minimum. Making the generic objects
  part of the state would require us to remove some constructors that are
  already in place and used by customers. For a concrete example, see *Annex 1*.
* We won’t be able to derive `Versionize`, so a manual implementation will
  be required (in case `Versionize` doesn’t add support for generics in the
  meantime). In *Annex 2*, there’s an example of such `Versionize`
  implementation (i.e. for the state of the RTC device).

### Testing

Translations between the supported versions are covered by unit tests. To make
sure we do not incur significant delays during translations, the performance is
measured using micro-benchmarks. We recommend customers to also test the
performance of translations at the VMM level.

### POCs

You can find two POCs that are integrating the proposal in the RTC device
implementation here:

- https://github.com/lauralt/vm-superio/tree/state_no_ev -> state doesn't
  include the `events` generic.
- https://github.com/lauralt/vm-superio/tree/state_ev -> state includes the
  `events` generic as well.


#### Annex 1

Example of updates needed to the interface when storing the generics in the
state as well

Let’s take the events example for the `Abc` component. With no state in place,
in order to create an `Abc` component, either `Abc::new()` or
`Abc::with_events(events)` (if the user wants an `Abc` with metric capabilities)
can be used. If we don’t save the state of the events in the `AbcState`
structure, then users can still use the same methods with the same behavior as
before. Those methods will call inside `from_state()` with the `Default`
implementation of the state object, as it was already seen in the example from
the proposal.  On the other hand, if the events would be saved in the state
structure, `with_events(events)` won’t make sense anymore.
`Abc::from_state(state: AbcState)` will have a different interface (because the
events will already be in state; there’s no need to have a separate `events`
parameter). In this case, the `events` parameter from `with_events(events)`
won’t be needed anymore.


#### Annex 2

A `Versionize` implementation of `RTCStateSer` that contains the events generic
as well:

```
impl<EV: RTCEvents + Versionize> Versionize for RTCStateSer<EV> {
    fn serialize<W: std::io::Write>(
        &self,
        writer: &mut W,
        version_map: &VersionMap,
        target_version: u16,
    ) -> VersionizeResult<()> {
        Versionize::serialize(&self.lr, writer, version_map, target_version)?;
        Versionize::serialize(&self.offset, writer, version_map, target_version)?;
        Versionize::serialize(&self.mr, writer, version_map, target_version)?;
        Versionize::serialize(&self.imsc, writer, version_map, target_version)?;
        Versionize::serialize(&self.ris, writer, version_map, target_version)?;
        Versionize::serialize(&self.events, writer, version_map, target_version)
    }

    #[inline]
    fn deserialize<R: std::io::Read>(
        reader: &mut R,
        version_map: &VersionMap,
        source_version: u16,
    ) -> VersionizeResult<Self> {
        let lr = Versionize::deserialize(reader, version_map, source_version)?;
        let offset = Versionize::deserialize(reader, version_map, source_version)?;
        let mr = Versionize::deserialize(reader, version_map, source_version)?;
        let imsc = Versionize::deserialize(reader, version_map, source_version)?;
        let ris = Versionize::deserialize(reader, version_map, source_version)?;
        let events = Versionize::deserialize(reader, version_map, source_version)?;

        Ok(RTCStateSer {
            lr,
            offset,
            mr,
            imsc,
            ris,
            events
        })
    }

    fn version() -> u16 {
        1
    }
}
```
