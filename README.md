# Boulder Release Process Docs & Demo

A repo for documenting and demonstrating the
[Boulder](https://github.com/letsencrypt/boulder) release process used
to move code from Github to
[staging](https://letsencrypt.org/docs/staging-environment/) and
[production](https://acme-v01.api.letsencrypt.org/).

## Goals

1. When we need to do a hotfix deploy, we can easily do one that
   contains only the fixes needed for hotfix, regardless of what changes
   have been committed in the meantime.

2. Any code we deploy meets our standard [code review
   process](https://github.com/letsencrypt/boulder/blob/main/CONTRIBUTING.md#review-requirements)
   and has passed all automated tests.

3. Any commit we deploy is eventually made available in the public
   boulder repo.

4. The hotfix release process differs as little as possible from the
   scheduled release process.

## Release Schedule

Boulder developers make a new release at the beginning of each week,
typically by 10am PST **Monday**. Operations deploys the new release to
the [staging
environment](https://letsencrypt.org/docs/staging-environment/) on
**Tuesday**, typically by 2pm PST. If there have been no issues
discovered with the release from its time in staging, then on
**Thursday** the operations team deploys the release to the production
environment.

Holidays, unexpected bugs, and other resource constraints may affect the
above schedule and result in staging or production updates being
skipped. It should be considered a guideline for normal releases but not
a strict contract.

## Making a Release

### Regular Releases

For regular staging releases, each week the Boulder dev team will tag
the current main commit with the current date, making a tag prefixed by
`"release-"` (e.g. `release-2018-01-22`). The tag name will always be
the date the release is tagged, not the date that it is pushed to
staging or production. If multiple releases are tagged on the same day
the tag will be given an incrementing letter suffix (e.g.
`release-2018-01-22a`)

### Hotfix Releases

#### When main is clean

When main is clean, and has no commits that we do not wish to include in
the hotfix release, then the regular release process is used. The hotfix
lands in main after the normal review process and a dev tags a new
release per "Regular Releases".

#### When main is dirty

When main is dirty, and has commits that are unrelated to the hotfix and
too risky to ship to prod we need to use an adjusted hotfix process. In
this case dev team will create a branch based on the current release tag
prefixed by `release-branch-`. Pull requests with the hotfix code will
be made against the release branch based on the normal Boulder
development process. Once the neccessary changes have been merged to the
release branch dev team will create a new hotfix release tag based on
that branch.

## Deploying Releases


When doing a release, Ops' tooling will verify that:

1. Travis shows that tests have passed for the commit at the planned
   release tag.

2. The planned release tag is an ancestor of the current main on GitHub
   **OR** the planned release tag is equal to the head of a branch named
   `release-branch-XXX`, and that branch contains a release-tagged
   commit, and all commits since then have followed the Boulder review
   process.

3. Each commit on the hotfix branch references a PR, and that PR was
   reviewed, and the PR page shows the final merged commit id.

## Prerequisites

* You must have a GPG key with signing capability:
  * [Checking for existing GPG
    keys](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/checking-for-existing-gpg-keys)

* If you don't have a GPG key with signing capability, create one:
    * [Generating a new local GPG
      key](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/generating-a-new-gpg-key)
    * [Generating a new Yubikey GPG
      key](https://support.yubico.com/hc/en-us/articles/360013790259-Using-Your-YubiKey-with-OpenPGP)

* The signing GPG key must be added to your GitHub account:
  * [Adding a new GPG key to your GitHub
    account](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/adding-a-new-gpg-key-to-your-github-account)

* `git` must be configured to call the correct GPG binary:
  * On most Linux platforms: `git config --global gpg.program gpg`
  * On macOS and some Linux platforms: `git config --global gpg.program
    gpg2`

* `git` must be configured to use the correct GPG key:
  * [Telling Git about your GPG
    key](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/telling-git-about-your-signing-key)

* Understand the [process for signing
  tags](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/signing-tags)

## Example

* On `2018-01-30`, @cpu submits [a pull
  request](https://github.com/letsencrypt/boulder-release-process/pull/1),
  proposing a feature to be merged to `main`.

* @jsha reviews and [approves the
  PR](https://github.com/letsencrypt/boulder-release-process/pull/1#pullrequestreview-92717219).
  The corresponding [travis
  build](https://travis-ci.com/letsencrypt/boulder-release-process/builds/64669844?utm_source=github_status&utm_medium=notification)
  passes, and so [@cpu merges the
  PR](https://github.com/letsencrypt/boulder-release-process/commit/500b4c3c010b7f8d97aca106aa76550b5cff72f0)
  into main.

* Later that day, @cpu cuts a release by making a [`release-2018-01-30`
  tag](https://github.com/letsencrypt/boulder-release-process/releases/tag/release-2018-01-30)
  from the [tip of
  `main`](https://github.com/letsencrypt/boulder-release-process/commit/500b4c3c010b7f8d97aca106aa76550b5cff72f0)
  at the time of tagging.

* Ops deploys `release-2018-01-30` some time later.

* After deploying `release-2018-01-30` a critical bug is discovered!
  Since `main` is clean, that is, has no commits since
  `release-2018-01-30` we don't want to ship with a hotfix, @cpu quickly
  prepares a [hotfix
  PR](https://github.com/letsencrypt/boulder-release-process/pull/2) to
  propose a hotfix change be merged into `main`.

* @jsha reviews and [approves the hotfix
  PR](https://github.com/letsencrypt/boulder-release-process/pull/2#pullrequestreview-92737652).
  The corresponding [travis
  build](https://travis-ci.com/letsencrypt/boulder-release-process/builds/64675859?utm_source=github_status&utm_medium=notification)
  passes, and so [@cpu merges the hotfix
  PR](https://github.com/letsencrypt/boulder-release-process/commit/4ed5f8181852237dd6cde45fbc63bd199f2d3ec9)
  into main.

* With the `main` branch ready with a hotfix, @cpu cuts a hotfix release
  by making a [`release-2018-01-30a`
  tag](https://github.com/letsencrypt/boulder-release-process/releases/tag/release-2018-01-30a)
  from the [tip of
  `main`](https://github.com/letsencrypt/boulder-release-process/commit/4ed5f8181852237dd6cde45fbc63bd199f2d3ec9)
  at the time of tagging.

* Ops deploys `release-2018-01-30a`, fixing the bug

* On `2018-01-31`, @cpu submits a [pull
  request](https://github.com/letsencrypt/boulder-release-process/pull/3)
  to propose a new feature be merged into `main`. 

* Later that day @jsha reviews and [approves the feature
  PR](https://github.com/letsencrypt/boulder-release-process/pull/3#pullrequestreview-93045304).
  The corresponding [travis
  build](https://travis-ci.com/letsencrypt/boulder-release-process/builds/64744142?utm_source=github_status&utm_medium=notification)
  passes, and so [@cpu merges the
  PR](https://github.com/letsencrypt/boulder-release-process/commit/e926651516f4c275740b68987481475495b55b65).


* Later on `2018-01-31` its discovered that the bug in
  `release-2018-01-30`, which was hotfixed with `release-2018-01-30a` is
  still broken! Uh oh! Since main is **dirty**, and contains a [feature
  commit](https://github.com/letsencrypt/boulder-release-process/commit/e926651516f4c275740b68987481475495b55b65)
  not present in `release-2018-01-30` or `release-2018-01-30a` the
  hotfix must be prepared in a separate branch. @cpu starts a new branch
  [`release-branch-2018-01-30a`](https://github.com/letsencrypt/boulder-release-process/tree/release-branch-2018-01-30a)
  by checking out `release-2018-01-30a` and pushing it as the release
  branch.

* In a new branch off of `release-branch-2018-01-30a` @cpu creates a
  hotfix and creates a [pull
  request](https://github.com/letsencrypt/boulder-release-process/pull/5)
  proposing the hotfix be merged into `release-branch-2018-01-30a`.

* @rolandshoemaker reviews and [approves the hotfix
  PR](https://github.com/letsencrypt/boulder-release-process/pull/5#pullrequestreview-93077656).
  The corresponding [travis
  build](https://travis-ci.com/letsencrypt/boulder-release-process/builds/64779641?utm_source=github_status&utm_medium=notification)
  passed, and so @cpu [merges the
  PR](https://github.com/letsencrypt/boulder-release-process/commit/9242e5ce9e96b5d0fdbc5140c9a6f9dd983e0273)
  into `release-branch-2018-01-30a`.

* @cpu then cuts a new release from `release-branch-2018-01-30a` by
  making a [`release-2018-01-31`
  tag](https://github.com/letsencrypt/boulder-release-process/releases/tag/release-2018-01-31)
  from the [tip of
  `release-branch-2018-01-30a`](https://github.com/letsencrypt/boulder-release-process/commit/4ed5f8181852237dd6cde45fbc63bd199f2d3ec9)
  at the time of tagging.

* Ops deploys `release-2018-01-31`, fixing the bug again without having
  to introduce unrelated code from `main`
