# Contributing to rust-vmm

Contributions to rust-vmm are welcome!

:hammer: But this file is still under construction. :hammer:

## Merging code in rust-vmm

To contribute to rust-vmm, you need to open a Pull Request (PR) against the
main branch of the repositories. rust-vmm uses the
[fork and pull development model](https://docs.github.com/en/github/collaborating-with-issues-and-pull-requests/about-collaborative-development-models).

We ask you to only open PRs from your fork branches. This helps us keep the
rust-vmm repositories clean, and not clutter them with PR branches.

### Developer Certificate of Origin

Please sign-off your commits in order to agree to the
[Developer Certificate of Origin](https://developercertificate.org/).

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
  chosen from the rust-vmm [gatekeepers](MAINTAINERS.md#gatekeepers) list.

Sometimes a selected reviewer does not have the available bandwidth, time or
expertise to do a proper review. In such cases a new reviewer must be assigned
from either the repository code owners list or the rust-vmm gatekeepers list.
It is up to the initially assigned reviewer, or one of the rust-vmm gatekeepers
if the former is not available, to find the new reviewer.

It is important to note that anyone outside the repository code owners or
the rust-vmm gatekeepers lists is very welcome to review any pending PR.
However, any resulting PR approval from an external reviewer will not count as
one of the 2 mandatory approvals for the PR to be merged.
