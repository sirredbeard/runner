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

### TODO

- [ ] I manually overrode `vswhere` behavior in dev.sh (the build script), which did not function properly. It is currently hardcoded to MSBuild tools from Visual Studio 2022. I'm not sure why vswhere can't find the Visual Studio 2022 MSBuild tools, I need to debug and clean that up.
- [ ] I need to finish automating builds and releases, including keeping modifications one commit ahead of upstream.

### Notes

* There are great forks of runner that add many additional features, the goal of this project is to track upstream as closely as possible with the minimum neccessary changes to support Windows Containers
* This is based in part on a [PR](https://github.com/actions/runner/pull/1801) by [Szymon Sobik](https://github.com/SS1823)
* All modifications to this fork will be squashed to a single commit to make syncing with upstream and visualizing diff from upstream simpler