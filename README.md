# Update PowerShell on Azure VMs

## Overview

This repository contains a GitHub Actions workflow to update PowerShell on Azure Virtual Machines (VMs). It uses a canary deployment strategy, first updating a set of Development VMs before proceeding to Production VMs.

## Workflow Description

The workflow is defined in `.github/workflows/update-powershell.yml` and is triggered manually using GitHub's `workflow_dispatch` event. The workflow performs the following tasks:

1. **Update Development VMs**:
   - Checks out the repository.
   - Authenticates to Azure using the credentials stored in GitHub secrets.
   - Runs a PowerShell script on a set of Development VMs to update PowerShell to the latest version.
   - If the update is successful, it proceeds to the Production stage.

2. **Update Production VMs**:
   - Checks out the repository.
   - Authenticates to Azure using the credentials stored in GitHub secrets.
   - Runs the same PowerShell script on Production VMs.

## How to Use

1. **Configure GitHub Secrets**:
   - Ensure you have the following secrets configured in your GitHub repository:
     - `AZURE_CREDENTIALS`: The Azure service principal credentials for authenticating to Azure.

2. **Update the Workflow Configuration**:
   - Edit the `.github/workflows/update-powershell.yml` file to include your specific resource groups and VM names.
   - Replace placeholder values `<dev-resource-group>`, `<dev-vm-name-1>`, `<prod-resource-group>`, `<prod-vm-name-1>`, etc., with actual names from your Azure environment.
   - Set the `requiredVersion` parameter to the desired version of PowerShell.

3. **Upload the PowerShell Script**:
   - Ensure the PowerShell script `Update-PowerShell.ps1` is located in the `scripts/` directory within your repository. This script is responsible for updating PowerShell on the VMs.

4. **Trigger the Workflow**:
   - Manually trigger the workflow from the GitHub Actions tab in your repository.

## PowerShell Script

The `Update-PowerShell.ps1` script is responsible for:
- Checking the current version of PowerShell on the VM.
- Verifying that there is a recent backup in the Azure Recovery Services Vault.
- Updating PowerShell to the specified version if necessary.

### Example `Update-PowerShell.ps1`

```powershell
param(
    [string]$requiredVersion,
    [string]$recoveryVaultName,
    [string]$resourceGroupName
)

# Function to get installed PowerShell version
function Get-InstalledPowerShellVersion {
    $version = (Get-Command pwsh.exe -ErrorAction SilentlyContinue).FileVersionInfo.ProductVersion
    if (-not $version) {
        $version = (Get-Command powershell.exe -ErrorAction SilentlyContinue).FileVersionInfo.ProductVersion
    }
    return $version
}

# Function to check backup status
function Check-BackupStatus {
    param (
        [string]$vaultName,
        [string]$resourceGroupName
    )

    # Placeholder for actual backup status check
    return $true  # Assume backup exists for demonstration
}

# Function to install the latest PowerShell version
function Install-LatestPowerShell {
    param (
        [string]$requiredVersion
    )

    # Check current version
    $currentVersion = Get-InstalledPowerShellVersion
    if ($currentVersion -lt $requiredVersion) {
        Write-Output "Updating PowerShell to version $requiredVersion"
        # Example installation command (change as needed)
        Install-Module -Name PowerShellGet -Force -AllowClobber
        Install-Module -Name PowerShellCore -Force -AllowClobber
    } else {
        Write-Output "PowerShell is already at the required version."
    }
}

# Main script logic
if (Check-BackupStatus -vaultName $recoveryVaultName -resourceGroupName $resourceGroupName) {
    Install-LatestPowerShell -requiredVersion $requiredVersion
} else {
    Write-Output "No recent backup found. Aborting update."
}
