# Contributing to rust-vmm

Contributions to rust-vmm are welcome!

:hammer: But this file is still under construction. :hammer:

## Merging code in rust-vmm

To contribute to rust-vmm, you need to open a Pull Request (PR) against the
main branch of the repositories. rust-vmm uses the
[fork and pull development model](https://docs.github.com/en/github/collaborating-with-issues-and-pull-requests/about-collaborative-development-models).

We ask you to only open PRs from your fork branches. This helps us keep the
rust-vmm repositories clean, and not clutter them with PR branches.

### PR workflow

After a PR is submitted against one of the rust-vmm repositories, it will be
tested and reviewed:

- The repository
  [continuous integration (CI)](https://github.com/rust-vmm/community#ci---wip)
  automatically tests the submitted changes.
- The PR changes will be [manually reviewed](#pr-review-and-approval). We
  expect the review comments to be addressed through the same PR.

A PR can be merged into the repository once all CI tests pass and at least 2
reviewers approved the submitted changes.

### PR review and approval

A rust-vmm PR automatically gets 2 reviewers assigned to it. Both reviewers
must approve the PR before it can be merged.

PR reviewers are selected as follows:

- If the repository has a `CODEOWNERS` file, both reviewers are randomly picked
  from there.
- If the repository does not have explicit code owners, both reviewers are
  chosen from the rust-vmm [gatekeepers](GATEKEEPERS.md) list.

Sometimes a selected reviewer does not have the available bandwidth, time or
expertise to do a proper review. In such cases a new reviewer must be assigned
from either the repository code owners list or the rust-vmm gate keepers list.
It is up to the initially assigned reviewer, or one of the rust-vmm gatekeepers
if the former is not available, to find the new reviewer.

It is important to note that anyone outside of the repository code owners or
the rust-vmm gate keepers lists is very welcome to review any pending PR.
However, any resulting PR approval from an external reviewer will not count as
one of the 2 mandatory approvals for the PR to be merged.

## Development

### Updating the rust-vmm-ci

All rust-vmm components are tested using the
[rust-vmm-ci](https://github.com/rust-vmm/rust-vmm-ci/) submodule.
We recommend component owners and contributors alike to update the
`rust-vmm-ci` at least one time per month. We expect it to continuously evolve:
more tests added, updates to the
[rust-vmm-container](https://github.com/rust-vmm/rust-vmm-container), and other
goodies.

To update the `rust-vmm-ci`, run the following commands starting from the
local directory that corresponds to the git repository where the submodule
needs to be updated.

1. Make sure the `rust-vmm-ci` submodule is initialized locally:

```bash
git submodule update --init --recursive
```

2. Get the commit sha of the current commit.

```bash
cd rust-vmm-ci
OLD_COMMIT=`git rev-parse HEAD`
```

3. Update rust-vmm-ci to the latest commit on main:

```bash
git pull https://github.com/rust-vmm/rust-vmm-ci/ main
```

4. Get the pretty print of commits (this helps us keep track of updates):

```bash
git log --abbrev-commit --pretty=oneline ${OLD_COMMIT}..HEAD
```

5. Add the submodule update to a new commit:

```bash
# Change the directory to the root directory of the repo
cd ..
git add rust-vmm-ci
git commit -s
```

For the commit message use the text from step 4, but remove any branch
information that it might contain. The text should only have a list of sha1
and a short description.

Example of commit message update:

```bash
commit 6c984917be09327cfbe4c72b92825dbed3477c81 (HEAD -> update_rust_vmm_ci)
Author: Andreea Florescu <fandree@amazon.com>
Date:   Tue Aug 11 12:00:59 2020 +0200

    updated rust-vmm-ci
    
    0fc8ced refactor test_benchmark.py
    741b894 checkout to PR branch before finishing test_bench
    645a5c3 test_bench: don't crash when no bench on master
    bd32544 Fetch origin in benchmark test
    35beb91 Fix commit message test
    53427aa benchmarks: add test that can run at every PR
    abd2c90 Add test for commit message format
    fe859f4 Update container image to v6
    75d7254 run cargo check on all features
    
    Signed-off-by: Andreea Florescu <fandree@amazon.com>

```
