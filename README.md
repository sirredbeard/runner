## This is a fork of runner that supports Windows Containers


### Prerequisites

* Windows 1803+
* Container support and Hyper-V enabled in Windows
* Docker installed on Windows, see [my guide here](https://boxofcables.dev/a-lightweight-windows-container-dev-environment/)

### Differences

* Supports Windows Containers on x86 and Arm
* Disables requirement for Windows Server, any version of Windows with Docker installed should work
* Upstream project GitHub Actions are removed or modified as needed

### TODO

- [ ] Manually overrode `vswhere` behavior in dev.sh (the build script), which did not function properly on MSBuild tools, and is currently hardcoded to MSBuild tools from VS2022. I'm not sure why vswhere can't find the VS2022 MSBuild tools, I need to debug and clean that up.
- [ ] I need to automate builds that include the helpful ./config.cmd and ./run.cmd scripts.

### Notes

* There are great forks of runner that add many additional features, the goal of this project is to track upstream as closely as possible with the minimum neccessary changes to support Windows Containers
* This is based in part on a [PR](https://github.com/actions/runner/pull/1801) by [Szymon Sobik](https://github.com/SS1823)
* All modifications to this fork will be squashed to a single commit to make syncing with upstream and visualizing diff from upstream simpler