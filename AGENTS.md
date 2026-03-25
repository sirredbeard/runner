# Agent Automation Guide for sirredbeard/runner

This document describes the automated workflow setup for maintaining this Windows Container fork of actions/runner.

## Overview

This fork maintains exactly **one commit ahead** of upstream actions/runner. All modifications are squashed into a single commit to make syncing and diffing with upstream simple.

## Repository Structure

### The One Commit Strategy

The entire fork is structured as:
```
upstream/main (actions/runner)
    ↓
[One Commit: "Add Windows Containers Support"]
    ↓
main (sirredbeard/runner)
```

This single commit contains:
- Windows Container support code changes
- Modified/removed GitHub Actions workflows
- Updated README
- Version file updates (including `releaseVersion` and `releaseNote.md`)

**Critical:** Any change to `releaseVersion` to trigger a release (e.g. when syncing to a new upstream tag) must be squashed into this single commit via `git commit --amend`, never added as a separate commit. Two commits above upstream is always wrong.

## Automated Workflows

### 1. Daily Upstream Sync (`merge_upstream.yml`)

**Trigger:** Daily at 2:00 AM UTC (via cron schedule)

**What it does:**
1. Checks for new upstream release tags (not commits - only v*.*.* tags)
2. If a new release is found:
   - Resets local main to the new upstream release tag
   - Cherry-picks the single Windows Container commit on top
   - Updates `releaseVersion` file to match upstream
   - Force-pushes to trigger release workflow
3. If no new release, does nothing

**Key files:**
- `.github/workflows/merge_upstream.yml`
- `releaseVersion` (used to trigger releases)

**Important:** This workflow only syncs on **release tags**, not every commit. This prevents unnecessary builds.

### 2. CI Workflow (`build.yml`)

**Trigger:** Push to main, pull requests

**What it does:**
- Builds win-x64 and win-arm64 binaries
- Runs L0 tests on x64 only (ARM64 runners not available on GitHub Actions)
- Creates packaged ZIP files
- Uploads artifacts

**Does NOT:**
- Build Docker images (moved to CD workflow to avoid chicken-and-egg problem)
- Create releases

### 3. CD Workflow (`release.yml`)

**Trigger:** Changes to `releaseVersion` file

**What it does:**
1. **Check job:** Validates version hasn't been released yet
2. **Build jobs:** Builds win-x64 and win-arm64
   - Computes SHA256 checksums
   - Uploads binary ZIPs and SHA256 files as artifacts
3. **Release job:** 
   - Downloads all artifacts
   - Validates SHA256 checksums match
   - Creates GitHub release with `gh` CLI
   - Includes formatted release notes from upstream
4. **Publish-image jobs:** Builds and publishes Windows Container images
   - Builds Windows Server Core (ltsc2025) based Docker images
   - Pushes to GitHub Container Registry (ghcr.io)
   - Separate jobs for windows/amd64 and windows/arm64
   - Tags with both version number and "latest"

**Critical:** Uses `gh release create` instead of deprecated `actions/create-release`. Each architecture's SHA256 is computed and validated before release.

## Build System

### Development Scripts

The build system uses upstream's `dev.sh` script without modifications. Key points:

- **vswhere:** Uses upstream's vswhere logic to find MSBuild - DO NOT hardcode paths
- **MSBuild:** Finds Visual Studio's MSBuild automatically via vswhere
- **.NET SDK:** Downloads and uses the same .NET SDK version as upstream (defined in upstream's scripts)
- **Architecture support:** win-x64 and win-arm64 (both use same build process)

### Build Process

1. Layout: `./dev layout Release win-x64`
   - Restores NuGet packages
   - Builds all .NET projects (Sdk, Runner.Common, Runner.Listener, Runner.Worker, Runner.PluginHost, Runner.Plugins)
   - Builds .NET Framework 4.8 project (RunnerService.csproj)
   - Copies outputs to `_layout/bin/`

2. Package: `./dev package Release win-x64`
   - Creates ZIP file from `_layout/`
   - Includes bin/, externals/, and root scripts
   - Output: `_package/actions-runner-win-{arch}-{version}.zip`

### Package Contents

A complete package includes:

**Root level (4 files):**
- config.cmd
- run.cmd
- run-helper.cmd.template
- run-helper.sh.template

**bin/ directory (~235 files):**
- Main executables: Runner.Listener.exe, Runner.Worker.exe, Runner.PluginHost.exe, RunnerService.exe
- DLLs: Runner.Common.dll, Runner.Sdk.dll, Runner.Plugins.dll, and all dependencies
- Scripts (8 files):
  - actions.runner.plist.template
  - actions.runner.service.template
  - darwin.svc.sh.template
  - installdependencies.sh
  - runsvc.sh
  - systemd.svc.sh.template
  - update.cmd.template
  - update.sh.template

**externals/ directory (4 files):**
- Node.js v20.20.0 and v24.13.0 (exe + lib for each)

## Auto-Update Redirect Mechanism

### How It Works

When a runner receives an update message from GitHub's backend, it checks the download URL:

1. **Detection:** Message contains URL from `actions/runner` releases
2. **Redirection:** URL is rewritten to point to `sirredbeard/runner` fork release with same version
3. **Validation:** Fork release SHA256 is fetched and validated before installation
4. **Transparent:** User sees no difference; update process is identical

### URL Transform Example

```
Upstream URL: https://github.com/actions/runner/releases/download/v2.332.0/actions-runner-win-x64-2.332.0.zip
              ↓
Fork URL:     https://github.com/sirredbeard/runner/releases/download/v2.332.0/actions-runner-win-x64-2.332.0.zip
```

### Implementation Details

- **SelfUpdater.cs** (legacy update flow):
  - Line 179-200: Intercepts `DownloadLatestRunner()` URL in `DownloadUrl` property
  - Lines 655-710: Provides `RedirectToForkRelease()` and `GetForkReleaseSHA256()` helpers
  - Uses `#if !DEBUG` guard to skip redirect in test builds

- **SelfUpdaterV2.cs** (modern update flow):
  - Line 57-85: Intercepts update in `SelfUpdate()` method before `DownloadLatestRunner()` call
  - Lines 580-630: Same redirect helper methods as SelfUpdater
  - Uses `#if !DEBUG` guard to skip redirect in test builds

- **Version Detection:** 
  - Extracts version from `AgentRefreshMessage.TargetVersion`
  - Detects architecture from `updateMessage.OS` or `_targetPackage.Platform`
  - Constructs fork URL: `https://github.com/sirredbeard/runner/releases/download/v{version}/actions-runner-win-{arch}-{version}.zip`

- **SHA256 Validation:**
  - Fetches `actions-runner-win-{arch}-{version}.zip.sha256` from fork releases
  - Parses SHA256 from file content
  - Validates downloaded package matches before installation

### Why `#if !DEBUG`?

The redirect logic is only active in RELEASE production builds:

- **DEBUG builds** (tests): Use original upstream URLs to verify test infrastructure
- **RELEASE builds** (production): Redirect to fork releases for automatic updates

This ensures:
- ✅ Unit tests work without mocking external URLs
- ✅ Production runners automatically update to fork releases
- ✅ Both update flows (legacy SelfUpdater and modern SelfUpdaterV2) behave identically

## Code Modifications

The single commit modifies these files for Windows Container support:

1. **src/Runner.Worker/ContainerOperationProvider.cs**
   - Removes Windows Server requirement checks
   - Enables Windows Container operations on any Windows edition

2. **src/Runner.Worker/Container/DockerCommandManager.cs**
   - Adjusts Docker command handling for Windows Containers

3. **src/Runner.Service/Windows/RunnerService.csproj**
   - Updates .NET Framework target from 4.7 to 4.8 for ARM64 compatibility

4. **src/Runner.Listener/Runner.cs**
   - Version output displays simple version number compatible with upstream patterns

5. **src/Runner.Listener/SelfUpdater.cs**
   - Intercepts update URLs from GitHub's backend messages
   - Redirects `actions/runner` URLs to `sirredbeard/runner` fork releases
   - Validates fork releases using SHA256 checksums before installation
   - Redirect logic only active in RELEASE builds (wrapped in `#if !DEBUG`)
   - Backward compatible with upstream runner behavior

6. **src/Runner.Listener/SelfUpdaterV2.cs**
   - Same redirect and validation logic as SelfUpdater for newer update message flows
   - Intercepts PackageMetadata messages and applies fork URL redirection
   - Validates SHA256 before downloading and extracting updated runner

7. **Workflow files:**
   - Deleted: Most upstream workflow files (CodeQL, dependency checks, bots, etc.)
   - Modified: build.yml (Windows-only), release.yml (Windows binaries + Docker), docker-publish.yml
   - Added: merge_upstream.yml (daily sync automation)

5. **images/Dockerfile**
   - Converted from Ubuntu-based to Windows Server Core (ltsc2025) based
   - Downloads runner from fork's releases instead of building
   - Supports both amd64 and arm64 architectures

6. **README.md**
   - Updated for Windows Container fork
   - Removed TODO items after automation completed

## Testing and Validation

### Pre-Release Validation

Before each release, the workflow:
1. Computes SHA256 for each binary ZIP
2. Uploads SHA256 as separate artifacts
3. Downloads both binary and SHA256 artifacts
4. Validates SHA256 matches before creating release

### Package Comparison

To verify fork matches upstream:
```powershell
# Download both
gh release download v2.331.0 --repo sirredbeard/runner --pattern "*.zip"
gh release download v2.331.0 --repo actions/runner --pattern "actions-runner-win-x64-*.zip"

# Compare structure, scripts, and executables
# All 8 scripts should have matching SHA256 hashes
# All 4 main executables should be present
# Package structure should be identical
```

## Maintenance Tasks

### When Upstream Releases

The automation handles this, but manual verification:
1. Check merge_upstream workflow ran successfully
2. Verify CI build completed
3. Verify CD released correct version
4. Download and spot-check one architecture against upstream

### Manual Sync (if needed)

```bash
git fetch upstream
git reset --hard upstream/main
git cherry-pick <windows-container-commit>
# Update releaseVersion and releaseNote.md to match upstream tag, then squash into the single commit
git add releaseVersion releaseNote.md
git commit --amend --no-edit
git push origin main --force-with-lease
```

**Important:** Do not run `git commit` (without `--amend`) after cherry-picking. All changes — including `releaseVersion` updates — must be folded into the single commit via `--amend`. If you end up with two commits above upstream, use `git reset --soft <upstream-sha>` and re-commit as one.

### Updating the Single Commit

To add changes to the single commit:
```bash
# Make your changes
git add <files>
git commit --amend --no-edit
git push origin main --force-with-lease
```

## Troubleshooting

### Build Failures

**MSBuild not found:**
- Don't hardcode MSBuild paths in dev.sh
- Let vswhere handle finding Visual Studio
- GitHub Actions runners have VS 2022 Build Tools installed

**.NET Framework 4.8 build fails:**
- Check Visual Studio components are installed
- Ensure vswhere is finding correct VS installation

**Test failures:**
- Fork skips tests on ARM64 (upstream behavior)
- L0 tests only run on x64

### Workflow Failures

**merge_upstream.yml fails:**
- Check if upstream has breaking changes
- May need to resolve cherry-pick conflicts manually
- Ensure GitHub token has write permissions

**release.yml fails on SHA256 validation:**
- SHA256 mismatch indicates build determinism issue
- Re-run build to see if it's transient

**Docker builds fail:**
- Windows Container builds require full PowerShell functionality
- Using Windows Server Core (ltsc2025) instead of nanoserver for better compatibility
- May require adjusting GitHub Actions runner configuration

## Key Learnings

### What Works

✅ Single commit ahead strategy - easy to maintain and visualize diffs
✅ Automated daily sync on release tags only
✅ SHA256 validation before release
✅ Using `gh` CLI instead of deprecated actions
✅ Keeping dev.sh unmodified from upstream
✅ Windows Container Docker image builds on GitHub Actions with Server Core base

### What Doesn't Work (Yet)

❌ L0 tests on ARM64 (GitHub Actions doesn't provide ARM64 Windows runners)
❌ Cross-compiling ARM64 on x64 GitHub runners (requires native ARM64 runner)

### Best Practices

1. **Minimal changes:** Only modify what's necessary for Windows Containers
2. **Stay with upstream patterns:** Use their build scripts, don't reinvent
3. **One commit discipline:** Always amend, never add commits — this includes `releaseVersion` updates for releases
4. **Test locally first:** Build and test both architectures locally before pushing
5. **Sync on releases:** Don't sync on every upstream commit, only tagged releases

## Version History

- **v2.332.0** - Auto-update redirection configuration
  - Implements fork auto-update redirection for transparent update flow
  - Adds SHA256 validation for fork releases before installation
  - Redirect logic guarded for production RELEASE builds only
  - Both SelfUpdater and SelfUpdaterV2 update flows redirected
  - All L0 tests passing (proxy, update behavior, etc.)

- **v2.331.0** - Complete automation with Windows Container Docker images
  - Switched Docker base from nanoserver to Windows Server Core (ltsc2025)
  - Successfully building and publishing Docker images to ghcr.io
  - Both amd64 and arm64 container images published
  - Verified package contents match upstream exactly (all scripts, executables, DLLs)
  - L0 tests simplified to run on x64 only (ARM64 runners not available)
  - Complete automation documentation in AGENTS.md
  - README overhauled to follow upstream style while clearly documenting changes

## Future Improvements

- [ ] Add native ARM64 Windows runner support when available on GitHub Actions
- [ ] Add package size validation (detect unexpected bloat)
- [ ] Add automated tests for Windows Container-specific functionality
