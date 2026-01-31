## This is a fork of runner that supports Windows Containers

### Prerequisites

* Windows 1803+
* Container support enabled
* Hyper-V and Virtual Machine Platform enabled, if using Hyper-V isolation
* Docker installed on Windows, see [my guide here](https://boxofcables.dev/a-lightweight-windows-container-dev-environment/)

### Differences from Upstream

* Supports Windows Containers on x86 and Arm.
* Disables the requirement for Windows Server. Any edition of Windows with Docker installed should work. However, to run containers in Hyper-V isolation, Windows Pro or Enterprise is required and nested virtualization must be available. [Read more](https://learn.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/hyperv-container) about Windows Container isolation.
* Upstream project GitHub Actions workflows are removed or modified.

### Releases

Binary releases are available in the [Releases](https://github.com/sirredbeard/runner/releases) section for:
* Windows x64
* Windows ARM64

Container images (based on Windows Server Core ltsc2025) are published to GitHub Container Registry (coming soon):
* `ghcr.io/sirredbeard/actions-runner:latest-amd64`
* `ghcr.io/sirredbeard/actions-runner:latest-arm64`
* `ghcr.io/sirredbeard/actions-runner:{version}-amd64`
* `ghcr.io/sirredbeard/actions-runner:{version}-arm64`

Releases track upstream [actions/runner](https://github.com/actions/runner) and are automatically synced daily.

### Notes

* There are great forks of runner that add many additional features, the goal of this project is to track upstream as closely as possible with the minimum neccessary changes to support Windows Containers
* This is based in part on a [PR](https://github.com/actions/runner/pull/1801) by [Szymon Sobik](https://github.com/SS1823)
* All modifications to this fork will be squashed to a single commit to make syncing with upstream and visualizing diff from upstream simpler