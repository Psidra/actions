# Reusable workflow: Build VS Code extension (.vsix)

Use this workflow to build a VS Code extension and either attach the .vsix to a GitHub Release or upload it as a CI artifact.

## Usage

### Release: attach .vsix to the GitHub Release
```yaml
name: Build and attach VSIX on release
on:
  release:
    types: [published]
jobs:
  build-vscode-extension:
    uses: Psidra/actions/.github/workflows/build-vscode-extension.yml@v1
    with:
      node_version: "20"
      package_manager: "npm" # or "pnpm" | "yarn"
      # build_script: "skip" # optionally skip build
      upload_mode: "release"
      working_directory: "." # or path to your extension
```

### PR CI: upload .vsix as an Actions artifact
```yaml
name: CI build (vsix)
on:
  pull_request:
    types: [opened, synchronize, reopened, edited]
jobs:
  build-vscode-extension:
    uses: Psidra/actions/.github/workflows/build-vscode-extension.yml@v1
    with:
      node_version: "20"
      package_manager: "npm"
      upload_mode: "artifact"
      artifact_name: "vsix"
      working_directory: "."
```

## Inputs
- node_version (string, default: "20") – Node version to setup
- package_manager (string, default: "npm") – one of npm | pnpm | yarn
- build_script (string, default: "npm run build") – set to "skip" to skip build
- vsix_output_glob (string, default: "*.vsix") – glob to find packaged files
- working_directory (string, default: ".") – directory of the extension
- upload_mode (string, default: "release") – "release" to attach to GitHub Release; "artifact" to upload as Actions artifact
- artifact_name (string, default: "vsix") – name of the uploaded artifact when upload_mode=artifact

## Notes
- Ensure your package.json includes required fields for `vsce package` (publisher, name, version, engines.vscode).
- If you consume this via `@v1`, after merging this change the `v1` tag should be updated to point to the latest commit to pick up the new inputs.

## Background
This enhancement helps avoid failures when a PR attempts to upload assets to a Release (which doesn't exist for pull_request events). For example, a PR build that tried to attach a .vsix to a Release failed quickly (see screenshot):

![image1](image1)