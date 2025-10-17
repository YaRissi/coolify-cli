# How to Release Coolify CLI

This guide explains the release process for the Coolify CLI.

## Prerequisites

- Write access to the `coollabsio/coolify-cli` repository
- All changes merged to the target branch (`v4.x`)
- All tests passing (`go test ./internal/...`)

## Release Process

### 1. Ensure All Changes Are Merged

All code changes should be merged to the target branch (`v4.x`). The version is automatically injected during the release process via GoReleaser.

```bash
git add cmd/root.go
git commit -m "chore: bump version to 1.x.x"
git push origin v4.x
```
### 2. Create a GitHub Release

1. Go to https://github.com/coollabsio/coolify-cli/releases/new
2. Click "Choose a tag" and create a new tag:
   - **Tag name**: `v1.x.x` (must start with `v`, e.g., `v1.2.3`)
   - **Target**: `v4.x` (or your target branch)
3. **Release title**: `v1.x.x` (same as tag name)
4. **Description**: Write release notes describing:
   - New features
   - Bug fixes
   - Breaking changes (if any)
   - Example:
     ```markdown
     ## What's New
     - Added support for database management
     - Improved error messages for API failures

     ## Bug Fixes
     - Fixed panic when config file is missing

     ## Breaking Changes
     - None
     ```
5. Click "Publish release"

### 3. Automated Build Process

Once you publish the release:

1. GitHub Actions automatically triggers the `release-cli.yml` workflow
2. GoReleaser automatically detects the version from the git tag
3. Version is injected into binaries via ldflags:
   - `internal/version.version` is set to the release tag (e.g., `v1.2.3`)
   - For snapshot/dev builds, `internal/version.versionPrerelease` includes commit hash
4. GoReleaser builds binaries for:
   - **Linux**: amd64, arm64
   - **macOS (Darwin)**: amd64, arm64
   - **Windows**: amd64, arm64
5. Binaries are automatically uploaded to the release
6. The release becomes available at:
   - GitHub: `https://github.com/coollabsio/coolify-cli/releases/tag/v1.x.x`
   - Install script: `curl -fsSL https://raw.githubusercontent.com/coollabsio/coolify-cli/main/scripts/install.sh | bash`
   - `go install`: `go install github.com/coollabsio/coolify-cli/coolify@v1.x.x`

### 4. Verify the Release

After the workflow completes (usually 2-5 minutes):

1. Check the release page has all platform binaries
2. Test the install script:
   ```bash
   curl -fsSL https://raw.githubusercontent.com/coollabsio/coolify-cli/main/scripts/install.sh | bash
   coolify version
   ```
3. Test the auto-update functionality:
   ```bash
   # If you have an older version installed
   coolify update
   coolify version  # Should show the new version
   ```
4. Verify the version matches your release

## Troubleshooting

### Build Failed
- Check the GitHub Actions logs at https://github.com/coollabsio/coolify-cli/actions
- Common issues:
  - Syntax errors in Go code
  - Test failures
  - GoReleaser configuration issues

### Version Not Updating
- The tag must start with `v` (e.g., `v1.2.3`, not `1.2.3`)
- GoReleaser automatically detects the version from the git tag
- Ensure the git tag exists and is correct: `git tag -l | grep v1.x.x`
- Check that the workflow has write permissions

### Install Script Not Finding New Version
- Wait a few minutes for GitHub's CDN to update
- Check that binaries were uploaded to the release
- Verify the tag format is correct (`v1.x.x`)

## Release Checklist

Before creating a release:

- [ ] All tests pass: `go test ./internal/...`
- [ ] Code is formatted: `go fmt ./...`
- [ ] Changes merged to `v4.x` branch
- [ ] Release notes prepared
- [ ] Determine the new version number (semantic versioning)

After creating a release:

- [ ] GitHub Actions workflow completed successfully
- [ ] All platform binaries are present on the release page
- [ ] Install script downloads the new version
- [ ] `coolify version` returns the correct version

## Configuration Files

The release process uses these configuration files:

- `.goreleaser.yml` - GoReleaser configuration with version injection via ldflags:
  - `-X internal/version.version={{ .Version }}` - Injects tag version (e.g., `v1.2.3`)
  - `-X internal/version.versionPrerelease=...` - For snapshot builds with commit hash
- `.github/workflows/release-cli.yml` - GitHub Actions workflow that triggers on releases
- `scripts/install.sh` - User-facing install script
- `internal/version/checker.go` - Version package with `GetVersion()` function
- `coolify/main.go` - Binary entry point for `go install` support

## Notes

- **Version Injection**: Version is automatically injected from the git tag at build time (via GoReleaser ldflags)
- No manual version updates needed in code - just create a git tag!
- The CLI has auto-update checking built-in (checks every 10 minutes via `GetVersion()`)
- Users can manually update with `coolify update`
- Install script supports version pinning: `bash install.sh v1.2.3`
- Releases are immutable - if you need to fix something, create a new patch version
- Development builds show version as "dev" unless compiled with ldflags
