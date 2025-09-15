# Psidra/actions

Centralized reusable GitHub Actions workflows for your repos.

## What's in here

- `reusable-release-please`: opens/updates Release PRs, creates tags and GitHub Releases.
- `reusable-semantic-pr`: enforces Conventional Commit PR titles.
- `reusable-commitlint`: lints commit messages on PRs.
- `reusable-build-vscode-extension`: builds a VS Code `.vsix` and attaches it to Releases.

All workflows are under `.github/workflows/` and are called with `workflow_call`.

## How to use from a repo

Pin to a tag (recommended). After you publish this repo, create a release `v1`, then reference `@v1`:

```yaml
# .github/workflows/pr-title.yml
name: PR title must be conventional
on:
  pull_request_target:
    types: [opened, edited, synchronize, reopened]
jobs:
  semantic-pr:
    uses: Psidra/actions/.github/workflows/semantic-pr.yml@v1
```

```yaml
# .github/workflows/commitlint.yml
name: Commitlint
on:
  pull_request:
    types: [opened, synchronize, reopened, edited]
jobs:
  commitlint:
    uses: Psidra/actions/.github/workflows/commitlint.yml@v1
```

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches: [main]
  workflow_dispatch:
jobs:
  release-please:
    uses: Psidra/actions/.github/workflows/release-please.yml@v1
```

```yaml
# .github/workflows/build-on-release.yml
name: Build and attach VSIX on release
on:
  release:
    types: [published]
jobs:
  build-vscode-extension:
    uses: Psidra/actions/.github/workflows/build-vscode-extension.yml@v1
    with:
      node_version: "20"
      package_manager: "npm"   # or "pnpm" / "yarn"
      # build_script: "skip"    # if you don't need a build step
```

## Notes

- Keep local Release Please workflows disabled to avoid duplicate runs.
- If you need per-repo overrides, pass inputs via `with:` at the caller.
- Publishing to the VS Code Marketplace requires `VSCE_PAT` and an additional step; this repo currently just packages and attaches the `.vsix` to the GitHub Release.