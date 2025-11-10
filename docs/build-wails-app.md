# Reusable workflow: Build Wails Application

Use this workflow to build cross-platform Wails v2 applications for Windows, macOS, and Linux, with optional code signing and notarization. The workflow builds all platform binaries in parallel using a matrix strategy and uploads them to a GitHub Release.

## Usage

### Basic usage: Build and attach to GitHub Release
```yaml
name: Build Wails App on Release
on:
  release:
    types: [published]
jobs:
  build-wails-app:
    uses: Psidra/actions/.github/workflows/build-wails-app.yml@v1
    with:
      app_name: "Tailmate"
      go_version: "1.21"
      wails_version: "v2"
      node_version: "20"
```

### With code signing and notarization
```yaml
name: Build Wails App on Release
on:
  release:
    types: [published]
jobs:
  build-wails-app:
    uses: Psidra/actions/.github/workflows/build-wails-app.yml@v1
    with:
      app_name: "Tailmate"
      go_version: "1.21"
      wails_version: "v2"
      node_version: "20"
      working_directory: "."
    secrets:
      WINDOWS_CERT_BASE64: ${{ secrets.WINDOWS_CERT_BASE64 }}
      WINDOWS_CERT_PASSWORD: ${{ secrets.WINDOWS_CERT_PASSWORD }}
      APPLE_CERT_BASE64: ${{ secrets.APPLE_CERT_BASE64 }}
      APPLE_CERT_PASSWORD: ${{ secrets.APPLE_CERT_PASSWORD }}
      APPLE_ID: ${{ secrets.APPLE_ID }}
      APPLE_TEAM_ID: ${{ secrets.APPLE_TEAM_ID }}
      APPLE_APP_PASSWORD: ${{ secrets.APPLE_APP_PASSWORD }}
```

## Inputs

- **app_name** (string, required) – Name of the Wails application
- **go_version** (string, default: "1.21") – Go version to use for building
- **wails_version** (string, default: "v2") – Wails version (v2 or specific version like v2.6.0)
- **node_version** (string, default: "20") – Node.js version for frontend build
- **working_directory** (string, default: ".") – Working directory of the Wails project

## Secrets

All secrets are optional. If provided, they enable code signing and notarization:

### Windows Code Signing
- **WINDOWS_CERT_BASE64** – Base64-encoded Windows code signing certificate (.pfx)
- **WINDOWS_CERT_PASSWORD** – Password for the Windows certificate

### macOS Code Signing and Notarization
- **APPLE_CERT_BASE64** – Base64-encoded Apple Developer certificate (.p12)
- **APPLE_CERT_PASSWORD** – Password for the Apple certificate
- **APPLE_ID** – Apple ID for notarization
- **APPLE_TEAM_ID** – Apple Team ID for notarization
- **APPLE_APP_PASSWORD** – App-specific password for Apple ID notarization

### GitHub Token
- **token** (optional) – GitHub token for uploading release assets (defaults to `GITHUB_TOKEN`)

## Platform Builds

The workflow builds for the following platforms in parallel:

### Windows
- Builds `.exe` executable for AMD64
- Optional Authenticode signing if certificate secrets are provided
- Generates SHA256 checksum

### macOS
- Builds `.app` bundle for AMD64 and ARM64 (Apple Silicon)
- Creates `.dmg` disk image
- Optional code signing and notarization if certificate secrets are provided
- Generates SHA256 checksum

### Linux
- Builds binary for AMD64
- Creates AppImage (if linuxdeploy is available)
- Creates `.deb` package
- Generates SHA256 checksum

## Artifact Naming Convention

All artifacts follow this naming convention:
```
{app_name}-{version}-{platform}-{arch}.{ext}
```

Examples:
- `Tailmate-1.0.0-windows-amd64.exe`
- `Tailmate-1.0.0-darwin-arm64.dmg`
- `Tailmate-1.0.0-linux-amd64.deb`
- `Tailmate-1.0.0-linux-amd64.AppImage`

## Version Detection

The workflow attempts to extract the version from your `wails.json` file. If not found, it defaults to "dev".

## Code Signing Setup

### Windows
1. Export your code signing certificate as a `.pfx` file
2. Encode it to base64: `base64 -i certificate.pfx -o certificate.txt`
3. Add `WINDOWS_CERT_BASE64` secret with the base64 content
4. Add `WINDOWS_CERT_PASSWORD` secret with the certificate password

### macOS
1. Export your Apple Developer certificate as a `.p12` file from Keychain Access
2. Encode it to base64: `base64 -i certificate.p12 -o certificate.txt`
3. Add `APPLE_CERT_BASE64` secret with the base64 content
4. Add `APPLE_CERT_PASSWORD` secret with the certificate password
5. Add `APPLE_ID` secret with your Apple ID email
6. Add `APPLE_TEAM_ID` secret with your Apple Team ID
7. Generate an app-specific password for your Apple ID at [appleid.apple.com](https://appleid.apple.com) (see "App-Specific Passwords" in your account settings)
8. Add `APPLE_APP_PASSWORD` secret with the app-specific password

## Notes

- Ensure your Wails project has a valid `wails.json` configuration file
- The workflow automatically installs platform-specific dependencies (GTK, WebKit2GTK for Linux)
- SHA256 checksums are generated for all artifacts and uploaded as `checksums.txt`
- Code signing is automatically enabled when the respective secrets are provided
- The workflow uses a matrix strategy to build all platforms in parallel, reducing build time
- If building fails for one platform, other platforms continue to build (fail-fast: false)

## Requirements

Your Wails project should:
- Have a valid `wails.json` file in the working directory
- Have all frontend dependencies properly configured
- Be compatible with Wails v2

## Background

This workflow simplifies the process of building and distributing cross-platform Wails applications. It handles:
- Multi-platform builds in parallel
- Platform-specific packaging formats
- Optional code signing and notarization
- Automatic version detection
- SHA256 checksum generation
- GitHub Release asset uploads

All artifacts are automatically attached to the GitHub Release that triggered the workflow.
