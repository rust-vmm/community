# Contributing to rust-vmm

Contributions to rust-vmm are welcome!

:hammer: But this file is still under construction. :hammer:

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

3. Update rust-vmm-ci to the latest commit on master:

```bash
git pull https://github.com/rust-vmm/rust-vmm-ci/ master
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
