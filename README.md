# .reusable

Provides reusable workflows for used by `blugnu` projects and any other go projects that find them useful.

To avoid changes in these workflows breaking the workflows of dependent projects it is recommended to use a release version tag ref in the project workflow:

```yml
uses: blugnu/.reusable/.github/workflows/module-release.yml@v0.1.0
```

Callable workflows provide self-contained steps used to compose higher order workflows in dependent projects, or via ready-to-use higher order `.reusable` workflows that are also provided.

## Examples

### Ex. 1 - A Go Module Repository

> The module repository uses the provided higher order `module-release.yml` callable workflow.  This provides a complete test and release workflow for a Go module.

```yml
# release.yml
name: release pipeline
on: push
jobs:
  release:
    uses: blugnu/.reusable/.github/workflows/module-release.yml@v0.1.0
    secrets: inherit
```

### Ex. 2 - A Non-Go Module Repository

> `git-log.yml` is used to obtain a semantic version from the git log of the current branch and the `release.yml` workflow to create a tag and release (when merged to master).

```yml
# release.yml
name: release pipeline
on: push
jobs:
  gitlog:
    uses: blugnu/.reusable/.github/workflows/git-log.yml@v0.1.0
    secrets: inherit

  release:
    if: ${{ github.ref == 'refs/heads/master' }}
    uses: blugnu/.reusable/.github/workflows/release.yml@v0.1.0
    needs:
      - gitlog
    with:
      version: ${{ needs.gitlog.outputs.semver }}
      createTag: ${{ needs.gitlog.outputs.isTagged == 'false' }}
```

