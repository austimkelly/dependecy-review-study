# dependecy-review-study
A sample to test the boundaries and limitations of GHAS Dependency Review workflow.
Dependency review action can be found at https://github.com/actions/dependency-review-action and does not require a GHAS license to use.

While the action is a great way to review dependencies and block disallowed licenses, it has some limitations that are not well documented. This repository is a sample to document those edges.

You can see the issues reported here in [this pull request](https://github.com/austimkelly/dependecy-review-study/pull/1) using [this workflow file](https://github.com/austimkelly/dependecy-review-study/blob/main/.github/workflows/dependency-review.yml).

# Limitations with Dependency Review

## Unknown Licenses

Copyleft licenses can be a big problem for some organizations. While these copy-left licenses are well-known (AGLP, GPL, etc.), it can often times be very difficult to identify the license of a package programmatically. For example, one report describing [how to protect yourself from litigation due to unexpected license agreements](https://blog.inedo.com/python/python-package-licenses/) suggests that 13.6% of packages on PyPi have a GPL-3 license. Additionally, Snyk researchers found that [10% of python packages don't have any license at all](https://snyk.io/blog/over-10-of-python-packages-on-pypi-are-distributed-without-any-license/)!.  This means that for organizations that do not allow copy-left licenses, they should:

* Block a PR when a copy-left license is introduced
* Block on a PR when a packages does not have a license so it can be inspected and explicitly allowed.

An example of an AGPL license that is declared but not properly configured in the Meta info section of PyPi is https://pypi.org/project/anarchy-bot/. In this case, the license type will not be discovered.s

### Limitation #1: Blocking Unknown licenses

Dependency review will flag unknown license, but there is no mechanism to block.

### Limitation #2: `allow-license` cannot be used with `deny-license`

It would be great if you could set a flag to block when unknown licenses are discovered. However, I thought it would work if I included both `allow-license` and `deny-license`. It does not. The action will fail with the following error: "You cannot specify both allow-licenses and deny-licenses". This should be clarified in the documentation.
     

## Blocking with `deny-packages`

**Limitation**: This one seems to be known (see [issue #627](https://github.com/actions/dependency-review-action/issues/627)), but if you want to explicitly block a package you need to include the lock file (e.g. poetry.lock) in the repository. If you don't have a lock file with version info, the action will flag the package is denied, but will not actually block the PR.

Still, even when adding in the lock file, the action emits a warning an not an error. This is not ideal since it is not clear that the package is being blocked. The console output shows it knows the package is denied, but it does not block the PR:

```bash
Denied
  Config: pkg:pypi/pycrypto
  Config: pkg:pypi/anarchy-bot
  Change: pycrypto@2.6.1 is denied
  Change: pkg:pypi/pycrypto@2.6.1 is denied
  Change: anarchy-bot@24.2.4 is denied
  Change: pkg:pypi/anarchy-bot@24.2.4 is denied
```

## `show-openssf-scorecard-levels` option is unknown

**Limitation**: This is minor since the default is `true`, but I was surprised to see that the option is unknown despite being in the README.