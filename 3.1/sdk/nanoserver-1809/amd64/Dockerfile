# escape=`

# Installer image
FROM mcr.microsoft.com/windows/servercore:1809 AS installer

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# Retrieve .NET Core SDK
ENV DOTNET_SDK_VERSION 3.1.100-preview1-014420

RUN Invoke-WebRequest -OutFile dotnet.zip https://dotnetcli.blob.core.windows.net/dotnet/Sdk/$Env:DOTNET_SDK_VERSION/dotnet-sdk-$Env:DOTNET_SDK_VERSION-win-x64.zip; `
    $dotnet_sha512 = '313a868cb1adfa703ee1875f47a9160286625d4dfe1fab105c9c94117859a3d1d55d9bf2af6892cdede1257ddbb25cd1589780f624b4bfe51adc4123c2a58e27'; `
    if ((Get-FileHash dotnet.zip -Algorithm sha512).Hash -ne $dotnet_sha512) { `
        Write-Host 'CHECKSUM VERIFICATION FAILED!'; `
        exit 1; `
    }; `
    `
    Expand-Archive dotnet.zip -DestinationPath dotnet; `
    Remove-Item -Force dotnet.zip

# Install PowerShell global tool
ENV POWERSHELL_VERSION=7.0.0-preview.4 `
    POWERSHELL_DISTRIBUTION_CHANNEL=PSDocker-DotnetCoreSDK-NanoServer-1809

RUN Invoke-WebRequest -OutFile PowerShell.Windows.x64.$ENV:POWERSHELL_VERSION.nupkg https://pwshtool.blob.core.windows.net/tool/$ENV:POWERSHELL_VERSION/PowerShell.Windows.x64.$ENV:POWERSHELL_VERSION.nupkg; `
    $powershell_sha512 = '1f0a57a210f2934fba68f710600c0b24a98dc0e2b291883b2f7cbb9801f59c831e0428267febe5b37b1fdb74af15a069ab28a95ab3d1760b6cf7026e9492b7d4'; `
    if ((Get-FileHash PowerShell.Windows.x64.$ENV:POWERSHELL_VERSION.nupkg -Algorithm sha512).Hash -ne $powershell_sha512) { `
        Write-Host 'CHECKSUM VERIFICATION FAILED!'; `
        exit 1; `
    }; `
    `
    \dotnet\dotnet tool install --add-source . --tool-path \powershell --version $ENV:POWERSHELL_VERSION PowerShell.Windows.x64; `
    Remove-Item -Force PowerShell.Windows.x64.$ENV:POWERSHELL_VERSION.nupkg; `
    Remove-Item -Path \powershell\.store\powershell.windows.x64\$ENV:POWERSHELL_VERSION\powershell.windows.x64\$ENV:POWERSHELL_VERSION\powershell.windows.x64.$ENV:POWERSHELL_VERSION.nupkg -Force

# SDK image
FROM mcr.microsoft.com/windows/nanoserver:1809

COPY --from=installer ["/dotnet", "/Program Files/dotnet"]

COPY --from=installer ["/powershell", "/Program Files/powershell"]

# In order to set system PATH, ContainerAdministrator must be used
USER ContainerAdministrator
RUN setx /M PATH "%PATH%;C:\Program Files\dotnet;C:\Program Files\powershell"
USER ContainerUser

# Enable detection of running in a container
ENV DOTNET_RUNNING_IN_CONTAINER=true `
    # Enable correct mode for dotnet watch (only mode supported in a container)
    DOTNET_USE_POLLING_FILE_WATCHER=true `
    # Skip extraction of XML docs - generally not useful within an image/container - helps performance
    NUGET_XMLDOC_MODE=skip

# Trigger first run experience by running arbitrary cmd
RUN dotnet help