# Code Covergae

1. [Overview](#overview)
2. [Details](#details)

## Overview

Code coverage is important for ensuring our integrated testing covers
the functionally of the code. The common project CI is set to reject
pull requests which cause the overall code coverage to fall. Therefore
it is important that any new functionality comes with tests to
exercise it.

## Details

The code coverage is controlled by a file
`coverage_config_x86_64.json` which usually sits in the root directory
of the repository. The file controls which files are excluded from
coverage calculations as well as the current coverage score. When the
CI is run the new score is compared to the stored score and if it
has changed to much it will cause the CI checks to fail and block the
merge.

If the coverage has dropped you will need to add some more tests to
cover any newly added functionality. If the coverage has increased
then please update the value in the config file and re-submit your
pull request.

### Multi-binary repositories

Repositories that build multiple binaries present a special challenge
when new binaries are added. The solution is to have separate code
coverage configurations in each binaries root directory. Then when a
new binary target is introduced it doesn't have to immediately meet the
current coverage for the rest of the repository.

### Coverage Targets

* Library code is expected to have at least 80% coverage
* Binary targets are expected to have at least 60% coverage

The target is lower for binaries due to the complexity of exercising
code in main(). However with careful design most main() code can be
split into testable functions. You should always strive for as
complete coverage as possible.
