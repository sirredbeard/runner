## This is a fork of runner that supports Windows Containers


### Prerequisites

* Windows running on x64, there are no Windows Containers for Arm at present
* Container support and Hyper-V enabled in Windows, though nested virtualization support is not required
* Docker installed on Windows, see [my guide here](https://boxofcables.dev/a-lightweight-windows-container-dev-environment/)

### Differences

* Supports Windows Containers on x86 and Arm
* Disables requirement for Windows Server, any version of Windows with Docker installed should work
* Built on .NET 4.8, updated from .NET 4.5 in upstream

### TODO

- [ ] Manually overrode `vswhere` behavior in dev.sh (the build script), which did not function properly on MSBuild tools, and is currently hardcoded to MSBuild tools from VS2022. I'm not sure why vswhere can't find the VS2022 MSBuild tools, I need to debug and clean that up.
- [ ] I need to automate builds that include the helpful ./config.cmd and ./run.cmd scripts.

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

### Note

Thank you for your interest in this GitHub repo, however, right now we are not taking contributions. 

We continue to focus our resources on strategic areas that help our customers be successful while making developers' lives easier. While GitHub Actions remains a key part of this vision, we are allocating resources towards other areas of Actions and are not taking contributions to this repository at this time. The GitHub public roadmap is the best place to follow along for any updates on features we’re working on and what stage they’re in.

We are taking the following steps to better direct requests related to GitHub Actions, including:

1. We will be directing questions and support requests to our [Community Discussions area](https://github.com/orgs/community/discussions/categories/actions)

2. High Priority bugs can be reported through Community Discussions or you can report these to our support team https://support.github.com/contact/bug-report.

3. Security Issues should be handled as per our [security.md](security.md)

We will still provide security updates for this project and fix major breaking changes during this time.

You are welcome to still raise bugs in this repo.
