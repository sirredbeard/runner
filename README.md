## This is a fork of runner that supports Windows Containers


### Prerequisites

* Windows running on x64, there are no Windows Containers for Arm at present
* Container support and Hyper-V enabled in Windows, though nested virtualization support is not required
* Docker installed on Windows, see [my guide here](https://boxofcables.dev/a-lightweight-windows-container-dev-environment/)

### Differences

* Supports Windows Containers on Windows on x64
* Built on .NET 4.8, updated from .NET 4.5 in upstream

### TODO

- [ ] Overrode `vswhere` behavior in dev.sh (the build script), which did not function properly on MSBuild tools, and is currently hardcoded to MSBuild tools from VS2022, not sure why vswhere can't find the VS2022 MSBuild tools, need to debug and clean that up
- [ ] Automate builds that include the helpful ./config.cmd and ./run.cmd scripts

### Notes

* There are great forks of runner that add many additional features, the goal of this project is to track upstream as closely as possible with the minimum neccessary changes to support Windows Containers
* This is based in part on a [PR](https://github.com/actions/runner/pull/1801) by [Szymon Sobik](https://github.com/SS1823)
* All modifications to this fork will be squashed to a single commit to make syncing with upstream and visualizing diff from upstream simpler

-------------------------------------

<p align="center">
  <img src="docs/res/github-graph.png">
</p>

# GitHub Actions Runner

[![Actions Status](https://github.com/actions/runner/workflows/Runner%20CI/badge.svg)](https://github.com/actions/runner/actions)

The runner is the application that runs a job from a GitHub Actions workflow. It is used by GitHub Actions in the [hosted virtual environments](https://github.com/actions/virtual-environments), or you can [self-host the runner](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/about-self-hosted-runners) in your own environment.

## Get Started

For more information about installing and using self-hosted runners, see [Adding self-hosted runners](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/adding-self-hosted-runners) and [Using self-hosted runners in a workflow](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/using-self-hosted-runners-in-a-workflow)

Runner releases:

![win](docs/res/win_sm.png) [Pre-reqs](docs/start/envwin.md) | [Download](https://github.com/actions/runner/releases)  

![macOS](docs/res/apple_sm.png)  [Pre-reqs](docs/start/envosx.md) | [Download](https://github.com/actions/runner/releases)  

![linux](docs/res/linux_sm.png)  [Pre-reqs](docs/start/envlinux.md) | [Download](https://github.com/actions/runner/releases)

## Contribute

We accept contributions in the form of issues and pull requests. The runner typically requires changes across the entire system and we aim for issues in the runner to be entirely self contained and fixable here. Therefore, we will primarily handle bug issues opened in this repo and we kindly request you to create all feature and enhancement requests on the [GitHub Feedback](https://github.com/community/community/discussions/categories/actions-and-packages) page. [Read more about our guidelines here](docs/contribute.md) before contributing.
