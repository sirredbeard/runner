<p align="center">
  <img src="docs/res/github-graph.png">
</p>

# GitHub Actions Runner - Windows Container Fork

[![Actions Status](https://github.com/sirredbeard/runner/workflows/Runner%20CI/badge.svg)](https://github.com/sirredbeard/runner/actions)

This is a fork of [actions/runner](https://github.com/actions/runner) that runs jobs in Windows Containers on x64 and ARM64.

Upstream only supports container jobs on Linux runners, and its Windows code path additionally requires Windows Server. This fork removes both restrictions. It tracks upstream as a single commit ahead, so the delta is always one `git show` away.

## Differences from upstream

* Container jobs run on any edition of Windows 1803 or later with Docker installed, not just Windows Server.
* Containers can run under [Hyper-V isolation](https://learn.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/hyperv-container) on Windows Pro and Enterprise with nested virtualization enabled.
* Builds and releases are Windows-only: `win-x64` and `win-arm64`. The Linux and macOS build jobs and the Linux container image are removed.
* Most upstream repository workflows (CodeQL, npm audit, stale bots, dotnet/node/buildx upgrade jobs) are removed and replaced with an upstream sync job, see [AGENTS.md](AGENTS.md).

### Code changes

* **src/Runner.Worker/ContainerOperationProvider.cs**: Removes the Windows Server registry check. Maps `/github/home` and `/github/workflow` to `C:\ghhome` and `C:\ghworkflow`, and uses `ping -t localhost` as the keepalive entrypoint in place of `tail -f /dev/null`.
* **src/Runner.Worker/Container/DockerCommandManager.cs**: Allows container operations on Windows and creates missing bind mount source directories before `docker run`.
* **src/Runner.Service/Windows/RunnerService.csproj**: Raises the non-ARM64 service target from .NET Framework 4.7 to 4.8. Upstream already targets 4.8 for `win-arm64`.
* **src/Runner.Listener/SelfUpdater.cs** and **SelfUpdaterV2.cs**: Redirect auto-update downloads to fork releases.

Everything else matches upstream, including the build scripts and the contents of the runner package.

## Requirements

* Windows 1803 or later, any edition
* Containers feature enabled
* Docker installed on Windows ([setup guide](https://boxofcables.dev/a-lightweight-windows-container-dev-environment/))
* For Hyper-V isolation: Windows Pro or Enterprise with Hyper-V and Virtual Machine Platform enabled

## Releases

Windows x64 and ARM64 packages are published to [Releases](https://github.com/sirredbeard/runner/releases) and track upstream versions.

Windows Container images are published to GitHub Container Registry, based on Windows Server Core ltsc2025:

* `ghcr.io/sirredbeard/actions-runner:latest-amd64`
* `ghcr.io/sirredbeard/actions-runner:latest-arm64`
* `ghcr.io/sirredbeard/actions-runner:2.336.0-amd64`
* `ghcr.io/sirredbeard/actions-runner:2.336.0-arm64`

## Auto-update

GitHub's backend sends update messages pointing at `actions/runner` release assets. In RELEASE builds, the runner rewrites that URL to the matching fork release:

```
From: https://github.com/actions/runner/releases/download/v{version}/actions-runner-win-{arch}-{version}.zip
To:   https://github.com/sirredbeard/runner/releases/download/v{version}/actions-runner-win-{arch}-{version}.zip
```

No configuration is needed. DEBUG builds keep the original upstream URLs.

Each release ships a `.sha256` asset next to its ZIP, and the runner validates the download against it before installing. If that asset cannot be fetched, the update proceeds without hash validation.

## Usage

Configuring and using self-hosted runners is unchanged from upstream:

* [Adding self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/adding-self-hosted-runners)
* [Using self-hosted runners in a workflow](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/using-self-hosted-runners-in-a-workflow)

## Support

This is a personal fork. Support is best effort.

File Windows Container issues here. File anything reproducible on an upstream runner at [actions/runner](https://github.com/actions/runner).

## Acknowledgments

* [actions/runner](https://github.com/actions/runner) by GitHub
* Windows Container support based in part on [PR #1801](https://github.com/actions/runner/pull/1801) by [Szymon Sobik](https://github.com/SS1823)
