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
