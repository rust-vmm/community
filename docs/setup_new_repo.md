# Set Up Repository

## Create Repository

Creating repositories is limited to the administrators of the rust-vmm
organization. The administrators are:

- [Andreea Florescu](https://github.com/andreeaflorescu)
- [Johnathan Bryce](https://github.com/jbryce)
- [Samuel Ortiz](https://github.com/sameo)

Before creating the repository you should make sure that the rust-vmm community
is onboard with creating the repository and that they agree with the name of
the crate.

To create a repository for a Rust component (crate), you can use
[this link](https://github.com/organizations/rust-vmm/repositories/new). Make
sure to select the
[`rust-vmm/crate-template`](https://github.com/rust-vmm/crate-template) under
*Repository Template*. This step is important because it pulls in the
rust-vmm-ci and adds the configuration required for publishing crates.

Once the repository is created,
[invite](https://docs.github.com/en/organizations/managing-access-to-your-organizations-repositories/managing-team-access-to-an-organization-repository)
the rust-vmm/gatekeepers team as the Admin of the crate. This step is required
such that the org level maintainers can also help with setting up the
repositories (i.e. enable branch protection or set up the CI). We typically
also add the person requesting the crate as an Admin. 

## Configure Repository

This configuration is in line with the rust-vmm
[contributing guidelines](https://github.com/rust-vmm/community/blob/main/CONTRIBUTING.md#merging-code-in-rust-vmm),
and it is necessary for ensuring that all rust-vmm crates keep the same quality
standard.

### Set Up Branch Protection

To set up branch protection go to the repository settings page. Under
*Branches* edit the following protection rules:
- Require pull request reviews before merging;
  - Required approving review: 2
  - Dismiss stale pull request approvals when new commits are pushed: enabled

- Require status checks to pass before merging: enabled
  - Require branches to be up to date before merging: enabled

The other configurations can stay the same.

*Note: If you cannot see the repository settings page, ask one of the
administrators to check the permissions of the repository.*

### Set Up CI

The rust-vmm components are tested using the
[rust-vmm-ci](https://github.com/rust-vmm/rust-vmm-ci) running on
[Buildkite](https://buildkite.com/).

Setting up the CI can only be done by the rust-vmm Buildkite account
administrators:
- [Alexandru Agache](https://github.com/alexandruag)
- [Alexandra Iordache](https://github.com/aghecenco)
- [Andreea Florescu](https://github.com/andreeaflorescu)
- [Laura Loghin](https://github.com/lauralt)
- [Samuel Ortiz](https://github.com/sameo)
- [Sebastien Boeuf](https://github.com/sboeuf)

To set up the CI, you need to:

1. Create a new [pipeline](https://buildkite.com/organizations/rust-vmm/pipelines/new).
   The pipeline should use the naming convention `${repo_name}-ci`.
2. In the Pipeline configuration use as *Git Repository URL* the HTTPS URL of
   the git repository for which you are setting the CI.
   This is very important because otherwise the Buildkite Agent will not be
   able to fetch the code and run the CI. There are no SSH keys stored on the
   instances running the CI.
3. When first setting up a pipeline the only command that needs to be run is
   the one that uploads the default rust-vmm-ci pipeline. In the
   *Commands to run* write the following command:

```bash
./rust-vmm-ci/.buildkite/autogenerate_pipeline.py | buildkite-agent pipeline upload
```

4. Under *Teams*, give access to the `rust-vmm-dev` team to the pipeline and
   then click on *Create Pipeline*

Below you can see a configuration example:

![](../img/buildkite_pipeline_example.png)

5. Once you click on `Create Pipeline`, Buildkite will take to a page for
   setting up the the GitHub Webhook. Follow the instructions on that page.
6. Trigger a new build from the Buildkite interface to make sure that the
   setup is correct.
7. Go to the pipeline settings page:
    - Under *General*, click on `Make this Pipeline Public`.
    - Under *Builds*, enable "Skip Intermediate Builds" and
      "Cancel Intermediate Builds".
    - Under *GitHub*:
        - Enable "Build pull requests from third-party forked repositories"
        - Update commit statuses -> Show blocked builds in GitHub as -> Pending
        - Don't forget to click on "Save GitHub Settings"
