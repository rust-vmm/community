# Community

## What Is rust-vmm?

rust-vmm is an open-source project that empowers the community to build custom
Virtual Machine Monitors (VMMs) and hypervisors. It provides a set of
virtualization components that any project can use to quickly develop
virtualization solutions while focusing on the key differentiators of their
product rather than re-implementing common components like KVM wrappers, virtio
devices and other VMM libraries.

The rust-vmm project is organized as a shared effort, shared ownership
open-source project that includes (so far) contributors from Alibaba, AWS,
Cloud Base, Crowdstrike, Intel, Google, Red Hat as well as individual
contributors.

Each virtualization component lives in a separate GitHub repository under the
rust-vmm organization. One repository corresponds usually to one
[Rust crate](https://doc.rust-lang.org/stable/book/ch07-01-packages-and-crates.html).

## Why rust-vmm?

- **Reduce code duplication**. The initial thought with rust-vmm was to create
  a place for sharing common virtualization components between two existing
  VMMs written in Rust:
  [CrosVM](https://chromium.googlesource.com/chromiumos/platform/crosvm/) and
  [Firecracker](https://github.com/firecracker-microvm/firecracker/). These
  two projects have similar code for calling KVM ioctls, managing the
  virtual machine memory, interacting with virtio devices and others. Instead
  of having these components live in each of the projects repositories, they
  can be shared through the rust-vmm project.
- **Faster development**. rust-vmm provides a base of virtualization components
  which are meant to be generic such that they can be consumed by other
  projects besides CrosVM and Firecracker. One example that is often mentioned
  is building a container specific VMM. By doing so, the container VMM can
  reuse most of the rust-vmm components and simply build the glue around them
  as well as an appropriate API for interacting with the container.
- **Security & Testability**. Having independent components makes fuzz testing
  easy to apply to each individual package. Our top priority is now
  [vm-virtio](https://github.com/rust-vmm/vm-virtio) as it provides a
  [virtio](https://www.oasis-open.org/committees/tc_home.php?wg_abbrev=virtio)
  device implementation. We want to keep a high standard in terms of testing
  as these virtualization packages are going to be used in production by
  multiple projects. Each component is individually tested with a set of
  common build time tests responsible for running unit tests and linters
  (including coding style checks), and computing the coverage. In the future
  we are planning to test the integration of the rust-vmm components by
  providing a reference VMM implementation.
- **Clean interface**. As crates are shared between multiple VMMs, the interface has to be
  flexible enough to be used by multiple VMMs. There are a few rounds of design
  reviews up to the point we are confident the interface is clean and reusable
  by other projects.

## Status of rust-vmm Components

Each rust-vmm crate lives in its own repository. Depending on where the
latest crate code is, the crate can be in one of 3 states:

1. `empty`: The crate repo exists but there is no code there yet. This
   means the crate was created after a community acknowledgement about its
   relevance to the project, but either no code has been submitted yet or
   the initial code base is being reviewed through a pull request.

1. `rust-vmm`: The code is a work in progress. Once it meets the production
   [requirements](#publishing-on-cratesio---requirements-list), the crate
   will be pushed to [crates.io](https://crates.io).

1. `crates.io`: The crate is considered to be production ready. It can be
   consumed from Rust's canonical crate registry:
   [crates.io](https://crates.io). The crate repository in rust-vmm
   is as a staging area until the next push to [crates.io](https://crates.io).
   In other words, the production ready version of the code lives in
   [crates.io](https://crates.io) while development happens in the rust-vmm
   git repo.

### empty

These are the empty repositories that have PRs waiting to be merged.

- [vmm-vcpu](https://github.com/rust-vmm/vmm-vcpu/): a hypervisor-agnostic
  abstraction for Virtual CPUs (vCPUs).

### rust-vmm

- [mshv](https://github.com/rust-vmm/mshv): workspace for Microsoft Hypervisor
  safe wrappers.
- [vm-allocator](https://github.com/rust-vmm/vm-allocator): abstractions for
  allocating resources needed by the VMM while running (such as GSI numbers,
  MMIO ranges).
- [vm-virtio](https://github.com/rust-vmm/vm-virtio): workspace for virtio
  devices and related helpers.
- [vmm-reference](https://github.com/rust-vmm/vmm-reference): reference VMM
  built solely with `rust-vmm` crates and minimal glue.
- [vhost-device](https://github.com/rust-vmm/vhost-device): A workspace for
  devices implemented using the
  [vhost-user-backend](https://github.com/rust-vmm/vhost-user-backend).

### crates.io

- [event-manager](https://crates.io/crates/event-manager): abstractions
  for implementing event based systems.
- [kvm-bindings](https://crates.io/crates/kvm-bindings): Rust FFI bindings
  to KVM generated using [bindgen](https://crates.io/crates/bindgen).
- [kvm-ioctls](https://crates.io/crates/kvm-ioctls): Safe wrappers over the
  KVM API.
- [linux-loader](https://crates.io/crates/linux-loader): parser and loader
  for vmlinux and bzImage images as well as some other helpers for kernel
  commandline.
- [seccompiler](https://crates.io/crates/seccompiler): Linux seccomp-bpf
  jailing utilities.
- [vfio-bindings](https://crates.io/crates/vfio-bindings):
  Rust FFI bindings for using the VFIO framework.
- [vfio-ioctls](https://crates.io/crates/vfio-ioctls):
  safe wrappers over the VFIO framework.
- [vhost](https://crates.io/crates/vhost): a crate to support vhost backend
  drivers for virtio devices.
- [vhost-user-backend](https://crates.io/crates/vhost-user-backend): provides
  a framework to implement vhost-user backend services
- [virtio-bindings](https://crates.io/crates/virtio-bindings): Rust FFI
  bindings to virtio kernel headers generated using
  [bindgen](https://crates.io/crates/bindgen).
- [virtio-queue](https://crates.io/crates/virtio-queue):
  virtio queue implementation.
- [vm-device](https://crates.io/crates/vm-device): a virtual machine device
  model crate.
- [vm-fdt](https://crates.io/crates/vm-fdt): a Flattened Device Tree writer.
- [vm-memory](https://crates.io/crates/vm-memory): abstractions over a
  virtual machine's memory.
- [vm-superio](https://crates.io/crates/vm-superio): emulation for legacy
  devices.
- [vm-superio-ser](https://crates.io/crates/vm-superio-ser): provides version
  aware serialization for the emulated devices in vm-superio.
- [vmm-sys-util](https://crates.io/crates/vmm-sys-util/): collection of
  modules providing helpers and utilities for building VMMs and hypervisors.

## Development

### Join Us

You can join our community on any of the following places:

* Join our
  [mailing list](http://lists.opendev.org/cgi-bin/mailman/listinfo/rust-vmm).
* Join our
  [Slack channel](https://join.slack.com/t/rust-vmm/shared_invite/enQtODAxMzA2ODIyMTc2LWRhYjIwZmQ0YzUxODJlMTRhZWU2ZDBjYmJiNzBmOWVmYjg4MjY5YWRjYjM0YzQ5YzgyMTBmYzNlMjMzYmZlODU).

### Adding a New Virtualization Component

If you have a proposal about a new rust-vmm component, please open a
[review request issue](https://github.com/rust-vmm/community/issues/new?assignees=&labels=&template=new-crate-request.md&title=Crate+Addition+Request).
The issue must provide the explanations and justifications behind the need for
a new component, highlighting the purpose of this new repository, how will it
be used by other VMMs or hypervisors, and other information that you find
relevant.

Discussions about the proposal will take place on the same issue, and once an
agreement is reached on the new crate name (this part is hard!) and design,
an empty repository will be created in the rust-vmm GitHub organization.

To add functionality you will need to send Pull Requests (PRs) against the
newly created repository. While we do not expect the code to be perfect from
the first try, we do enforce a high standard in terms of quality for all
existing repositories under rust-vmm.

### Publishing on crates.io - Requirements List

We consider crates published on [crates.io](https://crates.io) to be production
ready. In order to maintain a high security and quality standard, the following
list of requirements must be checked before having the first version published:

- **High level documentation**. This documentation should be added
  to `src/lib.rs` and it will end up being the home page of the crate on
  [docs.rs](https://docs.rs/). Include information about what this crate does,
  how can it be used in other projects, as well as usage examples. Check out
  the [kvm-ioctls docs](https://docs.rs/kvm-ioctls/0.1.0/kvm_ioctls/) for an
  example.
- **Documentation for the public interface**. Everything belonging to the
  public interface of the crate must be properly documented. Where applicable,
  the methods should also have
  [usage examples](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#making-useful-documentation-comments).
  The code examples are particularly useful because they are run on
  `cargo test`. It is a nice way to ensure that your comments and examples
  don't go out of sync with the code. To make sure we don't miss anything this
  rule is enforceable by using `#![deny(missing-docs)]` in `src/lib.rs`.
- **Unit Tests**.
- **Coverage**. We expect all crates to have coverage tests that don't allow
  the coverage to drop on new PRs.
- **Build Time Tests**. All components will eventually be tested using the
  same common [Buildkite pipeline](https://buildkite.com/docs/pipelines). This
  is still WIP. The tests include style checks and other lint checks, a
  coverage test, builds with various configurations and on different
  architectures and operating systems.
- **README.md**. The readme contains high-level information about the crate.
  The readme is the front page of the package on crates.io so ideally it also
  has minimal examples on how to use the crate.
  For in-depth design details, a DESIGN.md file can be created.
- **LICENSE**.
- **CODEOWNERS**.
- **Complete `Cargo.toml`**. Check the
  [`cargo` reference](https://doc.rust-lang.org/cargo/reference/publishing.html#before-publishing-a-new-crate)
  for the **requirements** that a crate must meet to be published to
  [crates.io](https://crates.io).

### Contributing

All the rust-vmm repositories accept contributions by a
[GitHub Pull Request (PR)](https://help.github.com/articles/using-pull-requests/).
For more details, please check the [contributing document](CONTRIBUTING.md).

### CI

We ensure that all rust-vmm components keep the same quality bar by using the
[rust-vmm-ci](https://github.com/rust-vmm/rust-vmm-ci/). The rust-vmm-ci is
built on top of [Buildkite](http://buildkite.com/), and added as a submodule
to all rust-vmm repositories. For more details, please check the [rust-vmm-ci
README](https://github.com/rust-vmm/rust-vmm-ci/).

### Versioning Crates - WIP

This section is under construction.

### Fuzz Testing - WIP

This section is under construction.
