# Crate Release

1. [Prerequisites](#prerequisites)
2. [Regular Release](#regular-release)
3. [Patch Release](#patch-release)
4. [Release of a crate from a workspace](#release-of-a-crate-from-a-workspace)

## Prerequisites

Releases can be created by all [rust-vmm maintainers](../../MAINTAINERS.md),
and the code owners of each repository.

## Regular Release

1. Prepare any last-minute changes in a pull request, if necessary.
2. Update the `CHANGELOG.md` file in the root of the crate's repository. The
   first paragraph should be titled with the version of the new release and
   followed by subparagraphs detailing what's added, changed and fixed.
   Example for releasing v1.2.0:

```md
# v1.2.0

## Added

- New amazing API for doing all the work.

## Changed

- Magic parameter type is now `u64`.

## Fixed

- Fixed #42, the worst bug ever.

# v1.1.0
...
```

3. Update the version field in Cargo.toml in the root of the crate's
   repository.
4. Add a commit with the changelog and toml updates in the release pull
   request.
5. Once the pull request is merged, create a tag. Use the new version's
   changelog section for the tag text. Don't forget to remove the #s here,
   otherwise those lines won't appear in the tag message.
   Example for releasing v1.2.0:

```bash
git tag -a v1.2.0
# Write the tag body (example below) and exit the editor
v1.2.0

Added

- New amazing API for doing all the work.

Changed

- Magic parameter type is now `u64`.

Fixed

- Fixed #42, the worst bug ever.
```

6. Push the tag to the upstream repository: `git push upstream --tags`. In this
   example, the upstream remote points to the original repository (not your
   fork).

7. Create as GitHub release. Go to the Releases page in the crate's repository
   and click Draft a new release (button on the right). In Tag version, pick
   the newly pushed tag. In Release title, write the tag name including v
   (example: v1.2.3). The description should be the new version's changelog
   section. Click Publish release.
8. Publish the new version to crates.io. To double-check what's being
   published, do a dry run first. Make sure your HEAD is on the release tag.

```bash
cargo publish --dry-run
cargo publish
```

9. If this is the first time you publish a release of that crate on crates.io
   don't forget to grant permissions to the rust-vmm gatekeepers team and all
   the code owners (as specified in the CODEOWNERS file of the repository being
   published). If you are a code owner, but not a gatekeeper, you won't be
   able to add the gatekeepers team, only a member of that team is allowed to.
   In this case, you have to ask a code owner who is also a gatekeeper to
   grant this permission, and in case none of the code owners is a gatekeeper,
   you will have to add as an owner a gatekeeper as well (with the second
   command provided below). That gatekeeper can then add the gatekeepers team
   as an owner.

```bash
cargo owner --add github:rust-vmm:gatekeepers
cargo owner --add codeowner-github-handle
```

## Patch Release

:memo: Patch releases differ because they're not created off the
upstream main branch, but instead started off a stable release, and published
from a different, dedicated branch.

1. Checkout the tag you're starting from and create a new upstream branch. In
   the snippet below, the upstream remote points to the original repository,
   and the origin remote to your fork.
   Example setup for v1.2.1, which will be v1.2.0 plus a fix:

```bash
git checkout v1.2.0
git checkout -b v1.2.1_release
git push upstream v1.2.1_release # Push the upstream branch.
 # Create a local branch. This is what you'll be working on.
git checkout -b local_v1.2.1_release
# The development branch will sit in your fork.
git push -u origin local_v1.2.1_release
```

2. Follow the steps 2-8 from the [Regular Release process](#regular-release).
   **Pay attention to the branch against which you open the PR**. PRs need to be
   open against the vX.Y.Z_release branch (v1.2.1_release in the example
   above).

## Release of a crate from a workspace

Crates that are part of a workspace are different in terms of what is needed
for publishing them, because they require the CHANGELOG, README, and license
files in the root of each of them. These files have to be part of the crate so
that they will be included when packaging the crate for publishing.
The order in which the crates from a workspace are released is also important.
The first ones that have to be published are the ones that have no dependencies
on crates from the same workspace. For example, the `virtio-device` crate
depends on `virtio-queue`, so `virtio-queue` has to be published first. It is
not allowed to publish a crate that has a `path` dependency, more details
[here](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#multiple-locations).
After the dependency crates are released, the version of these crates has to
be updated in the `Cargo.toml` files of the crates that depend on them.
So for the particular example, after you release `virtio-queue` with 0.x.0
version, you have to update this dependency in the `Cargo.toml` of
`virtio-device`, i.e.:

`virtio-queue = { version = "=0.x.0", path = "../virtio-queue" }`

Then, `virtio-device` can be published as well.

1. Create a separate `CHANGELOG.md`, if it doesn't already exist, in the root
   of the crate. Update the file as mentioned in step 2 from the
   [Regular Release process](#regular-release).
2. For the README, either create a separate one with information about that
   particular crate, or add a symlink to the `README.md` file from the root of
   the workspace. To create such symlink, use the following command:

```bash
ln <path_to_the_root_of_the_repository>/README \
<path_to_the_root_of_the_crate>/README
```

*Example:*

```bash
ln /home/vm-virtio/README.md /home/vm-virtio/crates/virtio-device/README.md
```

3. The license files should have a symlink as well. For that, run the following
   commands:

```bash
ln <path_to_the_root_of_the_repository>/LICENSE-APACHE \
<path_to_the_root_of_the_crate>/LICENSE-APACHE
ln <path_to_the_root_of_the_repository>/LICENSE-BSD-3-Clause \
<path_to_the_root_of_the_crate>/LICENSE-BSD-3-Clause
```

4. Update the version field in the `Cargo.toml` from the root of the particular
   crate.

5. If the crate has a `path` dependency, update that dependency in `Cargo.toml`
   with a `version` that is published on crates.io as explained in the
   introduction. This version should be the latest one released.

6. Commit the symlinks together with the `Cargo.toml` and `CHANGELOG.md`
   updates.

7. This step is identical with step 5 from the
   [Regular Release process](#regular-release), but with the mention that the
   tag should include the name of the crate, so that it is possible to
   differentiate between the same version of different crates, example:
   
```bash
git tag -a virtio-device-v0.1.0
```

8. Now you can continue with steps 6 and 7 from the
   [Regular Release process](#regular-release).

9. Publish the new version to crates.io. To double-check what's being
   published, do a dry run first. Make sure your HEAD is on the release tag,
   and you run the following commands from the root of the crate, and not of
   the workspace.

```bash
cargo publish --dry-run
cargo publish
```

10. This step is identical with step 9 from the
    [Regular Release process](#regular-release).
