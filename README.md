# Boulder Release Process Docs & Demo

A repo for documenting and demonstrating the
[Boulder](https://github.com/letsencrypt/boulder) release process used to move
code from Github to [staging](https://letsencrypt.org/docs/staging-environment/)
and [production](https://acme-v01.api.letsencrypt.org/).

## Goals

1. When we need to do a hotfix deploy, we can easily do one that contains only
   the fixes needed for hotfix, regardless of what changes have been committed
   in the meantime.

1. Any code we deploy meets our standard [code review
   process](https://github.com/letsencrypt/boulder/blob/master/CONTRIBUTING.md#review-requirements)
   and has passed all automated tests.

3. Any commit we deploy is eventually made available in the public boulder repo.

4. The hotfix release process differs as little as possible from the scheduled
   release process.

## Making a Release

### Regular Releases

For regular staging releases, each week the Boulder dev team will tag the
current master commit with the current date, making a tag prefixed by
`"release-"` (e.g. `release-2018-01-22`). The tag name will always be the date
the release is tagged, not the date that it is pushed to staging or production.
If multiple releases are tagged on the same day the tag will be given an
incrementing letter suffix (e.g. `release-2018-01-22a`)

### Hotfix Releases

#### When master is clean

When master is clean, and has no commits that we do not wish to include in the
hotfix release, then the regular release process is used. The hotfix lands in
master after the normal review process and a dev tags a new release per "Regular
Releases".

#### When master is dirty

When master is dirty, and has commits that are unrelated to the hotfix and too
risky to ship to prod we need to use an adjusted hotfix process. In this case
dev team will create a branch based on the current release tag prefixed by
`release-branch-`. Pull requests with the hotfix code will be made against the
release branch based on the normal Boulder development process. Once the
neccessary changes have been merged to the release branch dev team will create
a new hotfix release tag based on that branch.

## Deploying Releases


When doing a release, Ops' tooling will check that:

1. Travis shows that tests have passed for the commit at the planned release tag.

1. The planned release tag is an ancestor of the current master on GitHub **OR**
   the planned release tag is equal to the head of a branch named
   `release-branch-XXX`, and that branch contains a release-tagged commit, and
   all commits since then have followed the Boulder review process.

1. Each commit on the hotfix branch references a PR, and that PR was reviewed,
   and the PR page shows the final merged commit id.

## Example

* On `2018-01-30`, @cpu submits [a pull request](https://github.com/letsencrypt/boulder-release-process/pull/1), proposing a feature to be merged to `master`.

* @jsha reviews and [approves the PR](https://github.com/letsencrypt/boulder-release-process/pull/1#pullrequestreview-92717219). The corresponding [travis build](https://travis-ci.com/letsencrypt/boulder-release-process/builds/64669844?utm_source=github_status&utm_medium=notification) passes, and so [@cpu merges the PR](https://github.com/letsencrypt/boulder-release-process/commit/500b4c3c010b7f8d97aca106aa76550b5cff72f0) into master.

* Later that day, @cpu cuts a release by making a [`release-2018-01-30` tag](https://github.com/letsencrypt/boulder-release-process/releases/tag/release-2018-01-30) from the [tip of `master`](https://github.com/letsencrypt/boulder-release-process/commit/500b4c3c010b7f8d97aca106aa76550b5cff72f0) at the time of tagging.

* Ops deploys `release-2018-01-30` some time later.

* After deploying `release-2018-01-30` a critical bug is discovered! Since `master` is clean, that is, has no commits since `release-2018-01-30` we don't want to ship with a hotfix, @cpu quickly prepares a [hotfix PR](https://github.com/letsencrypt/boulder-release-process/pull/2) to propose a hotfix change be merged into `master`.

* @jsha reviews and [approves the hotfix PR](https://github.com/letsencrypt/boulder-release-process/pull/2#pullrequestreview-92737652). The corresponding [travis build](https://travis-ci.com/letsencrypt/boulder-release-process/builds/64675859?utm_source=github_status&utm_medium=notification) passes, and so [@cpu merges the hotfix PR](https://github.com/letsencrypt/boulder-release-process/commit/4ed5f8181852237dd6cde45fbc63bd199f2d3ec9) into master.

* With the `master` branch ready with a hotfix, @cpu cuts a hotfix release by making a [`release-2018-01-30a` tag](https://github.com/letsencrypt/boulder-release-process/releases/tag/release-2018-01-30a) from the [tip of `master`](https://github.com/letsencrypt/boulder-release-process/commit/4ed5f8181852237dd6cde45fbc63bd199f2d3ec9) at the time of tagging.

* Ops deploys `release-2018-01-30a`, fixing the bug

* On `2018-01-31`, @cpu submits a [pull request](https://github.com/letsencrypt/boulder-release-process/pull/3) to propose a new feature be merged into `master`. 

* Later that day @jsha reviews and [approves the feature PR](https://github.com/letsencrypt/boulder-release-process/pull/3#pullrequestreview-93045304). The corresponding [travis build](https://travis-ci.com/letsencrypt/boulder-release-process/builds/64744142?utm_source=github_status&utm_medium=notification) passes, and so [@cpu merges the PR](https://github.com/letsencrypt/boulder-release-process/commit/e926651516f4c275740b68987481475495b55b65).


* Later on `2018-01-31` its discovered that the bug in `release-2018-01-30`, which was hotfixed with `release-2018-01-30a` is still broken! Uh oh! Since master is **dirty**, and contains a [feature commit](https://github.com/letsencrypt/boulder-release-process/commit/e926651516f4c275740b68987481475495b55b65) not present in `release-2018-01-30` or `release-2018-01-30a` the hotfix must be prepared in a separate branch. @cpu starts a new branch [`release-branch-2018-01-30a`](https://github.com/letsencrypt/boulder-release-process/tree/release-branch-2018-01-30a) by checking out `release-2018-01-30a` and pushing it as the release branch.

* In a new branch off of `release-branch-2018-01-30a` @cpu creates a hotfix and creates a [pull request](https://github.com/letsencrypt/boulder-release-process/pull/5) proposing the hotfix be merged into `release-branch-2018-01-30a`.

* @rolandshoemaker reviews and [approves the hotfix PR](https://github.com/letsencrypt/boulder-release-process/pull/5#pullrequestreview-93077656). The corresponding [travis build](https://travis-ci.com/letsencrypt/boulder-release-process/builds/64779641?utm_source=github_status&utm_medium=notification) passed, and so @cpu [merges the PR](https://github.com/letsencrypt/boulder-release-process/commit/9242e5ce9e96b5d0fdbc5140c9a6f9dd983e0273) into `release-branch-2018-01-30a`.

* @cpu then cuts a new release from `release-branch-2018-01-30a` by making a [`release-2018-01-31` tag](https://github.com/letsencrypt/boulder-release-process/releases/tag/release-2018-01-31) from the [tip of `release-branch-2018-01-30a`](https://github.com/letsencrypt/boulder-release-process/commit/4ed5f8181852237dd6cde45fbc63bd199f2d3ec9) at the time of tagging.

* Ops deploys `release-2018-01-31`, fixing the bug again without having to introduce unrelated code from `master`
