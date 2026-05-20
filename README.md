param (
    [string]$VMName,
    [string]$ResourceGroup
)

# ==========================================
# AVD Disk Cleanup Runbook
# Keep Only Last 10 Days Data
# ==========================================

# Connect using Managed Identity
Connect-AzAccount -Identity

# Remote Script
$ScriptBlock = @'

Write-Output "========== Cleanup Started =========="

# ==========================================
# PATHS
# ==========================================

$InstallerPath = "C:\Windows\Installer"
$WindowsTemp   = "C:\Windows\Temp"

# ==========================================
# 1. CLEAN WINDOWS INSTALLER
# Keep only last 10 days files
# ==========================================

if (Test-Path $InstallerPath) {

    Get-ChildItem $InstallerPath -Force -File -ErrorAction SilentlyContinue |
    Where-Object {
        $_.LastWriteTime -lt (Get-Date).AddDays(-10)
    } |
    ForEach-Object {

        try {

            Remove-Item $_.FullName -Force -ErrorAction Stop

            Write-Output "Deleted Installer File: $($_.Name)"

        }
        catch {

            Write-Output "Skipped Installer File: $($_.Name)"

        }
    }
}

# ==========================================
# 2. CLEAN WINDOWS TEMP
# Keep only last 10 days files
# ==========================================

if (Test-Path $WindowsTemp) {

    Get-ChildItem $WindowsTemp -Recurse -Force -ErrorAction SilentlyContinue |
    Where-Object {
        $_.LastWriteTime -lt (Get-Date).AddDays(-10)
    } |
    ForEach-Object {

        try {

            Remove-Item $_.FullName -Recurse -Force -ErrorAction Stop

            Write-Output "Deleted Temp File: $($_.FullName)"

        }
        catch {

            Write-Output "Skipped Temp File: $($_.FullName)"

        }
    }
}

# ==========================================
# 3. USER TEMP CLEANUP
# ==========================================

Get-ChildItem "C:\Users" -Directory -ErrorAction SilentlyContinue |
ForEach-Object {

    $UserTemp = "$($_.FullName)\AppData\Local\Temp"

    if (Test-Path $UserTemp) {

        Get-ChildItem $UserTemp -Recurse -Force -ErrorAction SilentlyContinue |
        Where-Object {
            $_.LastWriteTime -lt (Get-Date).AddDays(-10)
        } |
        ForEach-Object {

            try {

                Remove-Item $_.FullName -Recurse -Force -ErrorAction Stop

                Write-Output "Deleted User Temp: $($_.FullName)"

            }
            catch {

                Write-Output "Skipped User Temp: $($_.FullName)"

            }
        }
    }
}

# ==========================================
# 4. DISM COMPONENT CLEANUP
# ==========================================

Write-Output "Running DISM Cleanup..."

DISM.exe /Online /Cleanup-Image /StartComponentCleanup /ResetBase /Quiet

# ==========================================
# 5. FREE SPACE OUTPUT
# ==========================================

$Drive = Get-PSDrive C

Write-Output "Free Space Available: $([math]::Round($Drive.Free/1GB,2)) GB"

Write-Output "========== Cleanup Completed =========="

'@

# Execute on VM
Invoke-AzVMRunCommand `
    -ResourceGroupName $ResourceGroup `
    -VMName $VMName `
    -CommandId 'RunPowerShellScript' `
    -ScriptString $ScriptBlock