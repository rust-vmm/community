# Crate Release

1. [Prerequisites](#prerequisites)
2. [Regular Release](#regular-release)
3. [Patch Release](#patch-release)

## Prerequisites

Releases can be created by all [rust-vmm maintainers](../../MAINTAINERS.md),
and the code owners of each repository.

## Regular Release

This section outlines the steps required to publish crates from both single
repositories (i.e. repos that contain a single crate) and larger workspace
repos that may host multiple crates (such as `vm-virtio`).

Crates that are part of a workspace have a couple of additional requirements,
such as the presence of separate `CHANGELOG.md` and `README.md` files (potentially
even licenses) in the root of each of them. The order in which the crates
from a workspace are released can be important. The first to be published
are the ones that have no dependencies on crates from the same
workspace. For example, the `virtio-device` crate depends on `virtio-queue`,
so `virtio-queue` has to be published first. Workspace crates may include
both path and version information for dependencies from the same repository.
The  former is used during testing to validate against the most recent changes
(and is discarded during the publishing process; more details
[here](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#multiple-locations)),
while the latter represents what ends up being used after a crate is published.

After the dependency crates are released, the version of these crates has to
be updated in the `Cargo.toml` files of the crates that depend on them.
For the previous example, after we release `virtio-queue` with
version `0.x.0`, we have to update this dependency in the `Cargo.toml` of
`virtio-device`, i.e.: `virtio-queue = { version = "=0.x.0", path = "../virtio-queue" }`.
Then, `virtio-device` can be published as well.

Please follow the steps described below to publish the latest version of a crate:

1. Prepare any last-minute changes in a pull request, if necessary.
2. Update the `CHANGELOG.md` file in the root of the crate's folder. The
   first paragraph should be titled with the version of the new release and
   followed by subparagraphs detailing what's added, changed and fixed.
   Example for releasing version `v1.2.0` of the `vm-awesome` crate:

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

3. Update the version field in the `Cargo.toml` file from the crate's root folder.
4. If the crate is part of a workspace and has a path dependency, update that
   dependency in `Cargo.toml` with a version that is published on `crates.io`
   as explained in the introduction. This version should be the latest one released.
5. Add a commit with the changelog and `Cargo.toml` updates in the release pull
   request.
6. Once the pull request is merged, create a tag. Use the new version's
   changelog section for the tag text. Don't forget to remove the #s here,
   otherwise those lines won't appear in the tag message.
   Example for releasing `v1.2.0`:

```bash
git tag -a vm-awesome-v1.2.0
# Write the tag body (example below) and exit the editor
v1.2.0

Added

- New amazing API for doing all the work.

Changed

- Magic parameter type is now `u64`.

Fixed

- Fixed #42, the worst bug ever.
```

7. Push the tag to the upstream repository: `git push upstream --tags`. In this
   example, the upstream remote points to the original repository (not your
   fork).

8. Create a GitHub release. Go to the Releases page in the crate's repository
   and click Draft a new release (button on the right). In Tag version, pick
   the newly pushed tag. In Release title, write the tag name including v
   (example: vm-awesome-v1.2.3). The description should be the new version's 
   changelog section. Click Publish release.
9. Publish the new version to crates.io. To double-check what's being
   published, do a dry run first. Make sure your HEAD is on the release tag.

```bash
cargo publish --dry-run
cargo publish
```

10. If this is the first time you publish a release of that crate on crates.io
   don't forget to grant permissions to the rust-vmm gatekeepers team and all
   the code owners (as specified in the `CODEOWNERS` file of the repository being
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
git checkout vm-awesome-v1.2.0
git checkout -b vm-awesome-v1.2.1_release
git push upstream vm-awesome-v1.2.1_release # Push the upstream branch.
 # Create a local branch. This is what you'll be working on.
git checkout -b local_vm-awesome-v1.2.1_release
# The development branch will sit in your fork.
git push -u origin local_vm-awesome-v1.2.1_release
```

2. Follow the steps 2-8 from the [Regular Release process](#regular-release).
   **Pay attention to the branch against which you open the PR**. PRs need to be
   open against the vX.Y.Z_release branch (vm-awesome-v1.2.1_release in the example
   above).
