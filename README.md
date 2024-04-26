# .reusable

Provides reusable workflows for used by `blugnu` projects and any other go projects that find them useful.

Callable workflows in this repository provide self-contained steps used to compose higher order workflows
in dependent projects; alternatively, complete higher order `.reusable` workflows that are also provided
for common use cases, such as Go module releases.

To avoid workflows being broken by changes to `.reusable` workflows, it is recommended to pin workflows to
a release version rather than `@master`:

```yml
uses: blugnu/.reusable/.github/workflows/pipeline.module-release.yml@v0.2.0
```

## Workflow Filenames

Github Action workflow files are required to be placed in the `.github/workflows` folder; this applies to
caller workflows in another repository as well as the called workflows in this repository. This creates two
problems:

1. workflows cannot be organised into folders for different types of workflow; and
2. filename collisions may occur (_that is, a workflow file having the same filename as a workflow file
   in the caller repository_).

To address these problems `.reusable` workflow files are named with a prefix that indicates the type of
workflow in the file (`job.` or `pipeline.`); this also reduces (_but does not entirely eliminate_) the
possibility of filename collisions.

## Releases and Versioning

- Releases are created using [GoReleaser](https://goreleaser.com/).
- Versioning of releases is derived from [conventional commits](https://www.conventionalcommits.org) by
the `git-log.yml` workflow.

### Releases: GoReleaser Configuration

Workflows that create releases (whether using `release.yml` or the higher-order `module-release.yml` workflow) **MUST**
provide a `.goreleaser.yml` configuration file in the root of the repository.

For projects that do not ship binaries (e.g. Go modules) the following minimal configuration is sufficient:

```yml
# .goreleaser.yml
version: 1
builds:
  skip: true
```

## Versioning: Conventional Commits

Semantic versioning is derived from the git log of the current branch by applying a minimum interpretation of the
[conventional commits](https://www.conventionalcommits.org) specification.

Any commit type may be used, but only `fix`, `feat`, and `refactor` commits are considered significant for versioning.

### Versioning Rules

- one or more `fix` or `refactor` commit increments the patch version;
- one or more `feat` commit increments the minor version and resets the patch version;
- one or more `fix!` or `feat!` commit increments the major version and resets the minor and patch versions;

Two exceptions to the conventional commits specification are enforced:

- `BREAKING CHANGE` commits or footers are NOT allowed.
- `refactor!` commits are NOT allowed.

In both cases use a `fix!` or `feat!` commit instead.

No more than one increment is applied; the version will be incremented by the highest significant commit type.
e.g. a log comprised of `fix`, `feat`, and `refactor` commits will result in a single minor version increment
and the patch version being set to zero.

<hr>

## Workflow Examples

### A Go Module Repository

> The module repository uses the provided higher order `module-release.yml` callable workflow.
> This provides a complete test and release workflow for a Go module.

```yml
# release.yml
name: release pipeline
on: push
jobs:
  release:
    uses: blugnu/.reusable/.github/workflows/pipeline.module-release.yml@v0.1.1
    secrets: inherit
```

### A Non-Go Module Repository

> `git-log.yml` is used to obtain a semantic version from the git log of the current branch and
> the `release.yml` workflow to create a tag and release (when merged to master).

```yml
# release.yml
name: release pipeline
on: push
jobs:
  gitlog:
    uses: blugnu/.reusable/.github/workflows/job.git-log.yml@v0.2.0
    secrets: inherit

  release:
    if: ${{ github.ref == 'refs/heads/master' }}
    uses: blugnu/.reusable/.github/workflows/job.release.yml@v0.2.0
    needs:
      - gitlog
    with:
      version: ${{ needs.gitlog.outputs.semver }}
      createTag: ${{ needs.gitlog.outputs.isTagged == 'false' }}
```
