# Windows Container runner image
FROM mcr.microsoft.com/windows/servercore:ltsc2025

ARG TARGETARCH
ARG RUNNER_VERSION

ENV RUNNER_MANUALLY_TRAP_SIG=1
ENV ACTIONS_RUNNER_PRINT_LOG_TO_STDOUT=1

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

WORKDIR /actions-runner

# Download and extract runner
RUN $arch = if ($env:TARGETARCH -eq 'amd64') { 'x64' } else { $env:TARGETARCH }; \
    $url = \"https://github.com/sirredbeard/runner/releases/download/v$env:RUNNER_VERSION/actions-runner-win-$arch-$env:RUNNER_VERSION.zip\"; \
    Write-Host \"Downloading: $url\"; \
    Invoke-WebRequest -Uri $url -OutFile runner.zip; \
    Expand-Archive -Path runner.zip -DestinationPath .; \
    Remove-Item runner.zip

CMD ["powershell", "-File", ".\\run.ps1"]
