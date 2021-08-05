# Crate Release

1. [Prerequisites](#prerequisites)
2. [Regular Release](#regular-release)
3. [Patch Release](#patch-release)

## Prerequisites

Releases can be created by all [rust-vmm maintainers](../../GATEKEEPERS.md),
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
   published, do a dryrun first. Make sure your HEAD is on the release tag.

```bash
cargo publish --dry-run
cargo publish
```

9. If this is the first time you publish a release on crates.io don't forget to
   grant permissions to the rust-vmm gatekeepers team and all the code owners
   (as specified in the CODEOWNERS file of the repository being published).

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
