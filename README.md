# Azure CLI on Windows

## Latest version

Download and install the newest Azure CLI release. When the installer prompts for permission to make changes to your system, select "Yes".

If Azure CLI is already present, running either the 32-bit or 64-bit installer will overwrite the existing installation.

The EXE and ZIP packages are designed for installing or updating Azure CLI on Windows. You do not need to uninstall the previous version before using the EXE installer, as it will update the current installation automatically.

> \[!IMPORTANT]
> After installation finishes, **close and reopen all terminal windows before using Azure CLI**.

On Windows, Azure CLI can be installed using EXE or ZIP packages and accessed via PowerShell or Command Prompt (`cmd.exe`). For Windows Subsystem for Linux (WSL), packages compatible with your Linux distribution are available. See the [main install page](*) for supported package managers or manual instructions for WSL.

The present version of Azure CLI is **2.75.0**. For details about the newest release, check the [release notes](*). To verify your installed version and check for updates, run [az version](*).

#### Specific version

If you require a particular Azure CLI version, adjust the version portion in the download URL.

To get the MSI for a specific release, replace `<version>` in the URLs:
`https://azcliprod.blob.core.windows.net/msi/azure-cli-<version>.msi` (32-bit) or
`https://azcliprod.blob.core.windows.net/msi/azure-cli-<version>-x64.msi` (64-bit).

For instance, to download the 32-bit MSI for version [2.51.0](*), use `https://azcliprod.blob.core.windows.net/msi/azure-cli-2.51.0.msi`. The 64-bit MSI URL would be `https://azcliprod.blob.core.windows.net/msi/azure-cli-2.51.0-x64.msi`.

Available releases are documented in the [Azure CLI release notes](*). The 64-bit MSI has been available since version [2.51.0](*).

## Run the Azure CLI

**After installation, ensure you close and reopen all terminal windows.** Launch Azure CLI by using the `az` command from PowerShell or Command Prompt.

Before running any Azure CLI commands, log in to your Azure account. To sign in interactively, run:

```azurecli
az login
```

For more details on authentication, see [Sign into Azure with Azure CLI](*).

A common initial command is to review your active subscription:

```azurecli
az account show
```

## Troubleshooting installation

### PATH variable not set

This issue frequently occurs if the terminal has not been restarted after installation. Simply close and reopen all terminal windows.

### Proxy blocks connection

If your proxy prevents the MSI download, verify that proxy settings are correctly configured. On Windows 11, adjust via `Settings > Network & Internet > Proxy`. For managed environments, contact your administrator for specific configurations.

> \[!IMPORTANT]
> Proxy settings must allow Azure CLI to access Azure services through PowerShell or Command Prompt. In PowerShell, run:

```powershell
(New-Object System.Net.WebClient).Proxy.Credentials = `
[System.Net.CredentialCache]::DefaultNetworkCredentials
```

Ensure your proxy permits HTTPS access to:

* `https://aka.ms/`
* `https://azcliprod.blob.core.windows.net/`

Additional guidance is provided at [Work behind a proxy](*) in the Azure CLI troubleshooting documentation.

### Slow response times

See [Migrate to 64-bit Azure CLI](*) for performance enhancements.

## Enable tab completion in PowerShell

Tab completion (“Azure CLI completers”) helps by suggesting commands, parameters, and some parameter values when pressing <kbd>Tab</kbd>.

Tab completion is enabled by default in Azure Cloud Shell and many Linux distributions. From Azure CLI version 2.49 onward, you can activate tab completion in PowerShell as follows:

1. Open or create your PowerShell profile at `$PROFILE`. The simplest approach is `notepad $PROFILE`. More info can be found at [How to create your profile](*) and [Profiles and execution policy](*).
2. Add this snippet to your PowerShell profile:

```powershell
Register-ArgumentCompleter -Native -CommandName az -ScriptBlock {
    param($commandName, $wordToComplete, $cursorPosition)
    $completion_file = New-TemporaryFile
    $env:ARGCOMPLETE_USE_TEMPFILES = 1
    $env:_ARGCOMPLETE_STDOUT_FILENAME = $completion_file
    $env:COMP_LINE = $wordToComplete
    $env:COMP_POINT = $cursorPosition
    $env:_ARGCOMPLETE = 1
    $env:_ARGCOMPLETE_SUPPRESS_SPACE = 0
    $env:_ARGCOMPLETE_IFS = "`n"
    $env:_ARGCOMPLETE_SHELL = 'powershell'
    az 2>&1 | Out-Null
    Get-Content $completion_file | Sort-Object | ForEach-Object {
        [System.Management.Automation.CompletionResult]::new($_, $_, "ParameterValue", $_)
    }
    Remove-Item $completion_file, Env:\_ARGCOMPLETE_STDOUT_FILENAME, Env:\ARGCOMPLETE_USE_TEMPFILES, Env:\COMP_LINE, Env:\COMP_POINT, Env:\_ARGCOMPLETE, Env:\_ARGCOMPLETE_SUPPRESS_SPACE, Env:\_ARGCOMPLETE_IFS, Env:\_ARGCOMPLETE_SHELL
}
```

3. To show all options in a menu, append `Set-PSReadlineKeyHandler -Key Tab -Function MenuComplete` to your profile.

## Update the Azure CLI

From version [2.11.0](*), Azure CLI includes a built-in command to update itself to the latest release:

```azurecli
az upgrade
```

This also updates all installed extensions by default. For additional options, see the [command reference page](*). For versions prior to [2.11.0](*), reinstall Azure CLI as described in [Install the Azure CLI](*).

If using the ZIP package, remove the old folder and extract the new version into the *same directory*.

## Migrate to 64-bit Azure CLI

Since version 2.51.0, a 64-bit MSI installer is available and recommended for improved performance.

To migrate to the 64-bit Azure CLI:

1. Check your current Azure CLI version and installed extensions with `az --version`.
2. Extensions must be reinstalled. Backup the existing extension folder `%userprofile%\.azure\cliextensions` by renaming it if you may revert to 32-bit later. The folder will be recreated during extension reinstallation.
3. Download and install the latest 64-bit MSI as explained in [Install or update](*). The 32-bit MSI will be removed automatically.
4. Reinstall extensions using `az extension add --name <extension> --version <version>`. Alternatively, Azure CLI will prompt to install missing extensions on first use. More info at [How to install extensions](*).

If any problems occur after migration, uninstall the 64-bit version and reinstall the 32-bit MSI. Restore or rename your backed-up extensions folder if needed.

## Uninstall

If you choose to uninstall Azure CLI, we welcome your feedback. Before removal, run `az feedback` to submit suggestions or report issues. Our aim is to improve the Azure CLI experience. To report bugs, consider [filing a GitHub issue](*).

Uninstall Azure CLI via Windows "Apps and Features":

| Platform      | Instructions                                           |
| ------------- | ------------------------------------------------------ |
| Windows 11    | Start > Settings > Apps > Installed apps               |
| Windows 10    | Start > Settings > Apps > Apps & Features              |
| Windows 8 / 7 | Start > Control Panel > Programs > Uninstall a program |

Locate **Azure CLI** in the list. It appears as **Microsoft CLI 2.0 for Azure**. Select it and click `Uninstall`.

## Remove data

If you do not plan to reinstall Azure CLI, delete its cached data at:

* `C:\Users\<username>\.azure\msal_token_cache.bin`
* `C:\Users\<username>\.azure\msal_token_cache.json`
