# Update Rust Toolchain

The Rust version used for running the tests in rust-vmm crates is fixed such
that toolchain updates can only happen when the version is explicitly updated.

The Rust version is specified in the
[rust-vmm-container](https://github.com/rust-vmm/rust-vmm-container), and the
rust-vmm-container version is specified in
[rust-vmm-ci](https://github.com/rust-vmm/rust-vmm-ci). While rust-vmm-ci
does not declare a version itself, in the rust-vmm crates we are specifying
which commit from rust-vmm-ci to use for running the CI.

Thus, to update the Rust toolchain (or for that matter any other tools in the
CI), you need to go through the following updates:

1. Update the Rust toolchain version in the rust-vmm-container
[Dockerfile](https://github.com/rust-vmm/rust-vmm-container/blob/main/Dockerfile).
Open a PR with your change. Once the PR is approved and merged, the merge
will automatically trigger the publishing of a new container version. For more
details about how this works, you can check the rust-vmm-container
[README](https://github.com/rust-vmm/rust-vmm-container).
2. Update the rust-vmm-container version in the rust-vmm-ci
[autogenerate script](https://github.com/rust-vmm/rust-vmm-ci/blob/c2f8c93e3796d8b3ea7dc339fad211457be9c238/.buildkite/autogenerate_pipeline.py#L63).
and open a PR with your change. Once this is merged, you can go ahead with
updating the rust-vmm-ci submodule.
3. Update the rust-vmm-ci for the target rust-vmm repository. NOTE: the
rust-vmm-ci submodule is automatically updated using Dependabot
every week. If for any reason you need this update faster than the default
schedule, you can follow the runbook for manually updating rust-vmm-ci.
