<p align="center">
  <img src="docs/res/github-graph.png">
</p>

# GitHub Actions Runner - Windows Container Fork

[![Actions Status](https://github.com/sirredbeard/runner/workflows/Runner%20CI/badge.svg)](https://github.com/sirredbeard/runner/actions)

This is a fork of [actions/runner](https://github.com/actions/runner) that adds support for **Windows Containers on both x86 and ARM64 architectures**.

## Key Differences from Upstream

This fork makes the following changes:

1. **Windows Container Support**: Removes the requirement for Windows Server. The runner now works in Windows Containers on any edition of Windows with Docker installed (Windows 10/11 Pro, Enterprise, or Server).

2. **Hyper-V Isolation**: Supports running containers in Hyper-V isolation on Windows Pro/Enterprise with nested virtualization enabled. [Read more about Windows Container isolation](https://learn.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/hyperv-container).

3. **Architecture Support**: Provides pre-built releases for both Windows x64 and ARM64.

4. **Automated Syncing**: Automatically syncs with upstream releases daily, maintaining a single commit ahead of upstream for easy diffing and tracking.

### Code Changes

All modifications are consolidated into a single commit for maintainability:

- **src/Runner.Worker/ContainerOperationProvider.cs**: Removes Windows Server edition checks
- **src/Runner.Worker/Container/DockerCommandManager.cs**: Adjusts Docker command handling for Windows Containers
- **src/Runner.Service/Windows/RunnerService.csproj**: Updates .NET Framework target to 4.8 for ARM64 compatibility

## Get Started

### Prerequisites

* Windows 1803 or later
* Container support enabled
* Docker installed on Windows ([setup guide](https://boxofcables.dev/a-lightweight-windows-container-dev-environment/))
* For Hyper-V isolation: Windows Pro/Enterprise with Hyper-V and Virtual Machine Platform features enabled

### Binary Releases

Pre-built runner binaries are available for:

* **Windows x64**: [Download from Releases](https://github.com/sirredbeard/runner/releases)
* **Windows ARM64**: [Download from Releases](https://github.com/sirredbeard/runner/releases)

Releases track upstream [actions/runner](https://github.com/actions/runner) versions and are published automatically when new upstream releases are detected.

### Container Images

Windows Container images are available at GitHub Container Registry:

* `ghcr.io/sirredbeard/actions-runner:latest-amd64`
* `ghcr.io/sirredbeard/actions-runner:latest-arm64`
* `ghcr.io/sirredbeard/actions-runner:2.331.0-amd64`
* `ghcr.io/sirredbeard/actions-runner:2.331.0-arm64`

Images are based on Windows Server Core (ltsc2025).

## Usage

For information about configuring and using self-hosted runners, see:
- [Adding self-hosted runners](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/adding-self-hosted-runners)
- [Using self-hosted runners in a workflow](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/using-self-hosted-runners-in-a-workflow)

## Upstream Compatibility

This fork maintains minimal changes to stay compatible with upstream:

* Uses upstream's build scripts without modification
* Follows upstream's .NET version and dependencies
* Maintains identical package structure (all scripts, binaries, and tools match upstream exactly)
* Automatically syncs with new upstream releases

## Contributing

This is a personal fork focused on maintaining Windows Container support. The goal is to track upstream as closely as possible with minimal necessary changes.

For issues with the base runner, please report them to [actions/runner](https://github.com/actions/runner).

For Windows Container-specific issues, please open an issue in this repository.

## Acknowledgments

* Upstream project: [actions/runner](https://github.com/actions/runner) by GitHub
* Windows Container support based in part on [PR #1801](https://github.com/actions/runner/pull/1801) by [Szymon Sobik](https://github.com/SS1823)
* All modifications maintained in a single commit for transparency and ease of maintenance

## Automation

This fork uses automated workflows to:
- Check for new upstream releases daily
- Build Windows x64 and ARM64 binaries
- Validate packages with SHA256 checksums
- Publish releases and container images automatically

See [AGENTS.md](AGENTS.md) for detailed automation documentation.