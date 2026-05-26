# ==========================================
# FSLogix Stale Profile Cleanup Script
# Azure Virtual Desktop (AVD)
# ==========================================

param (

    [Parameter(Mandatory=$false)]
    [string]$ProfileShare = "\\storageaccount.file.core.windows.net\fslogix",

    [Parameter(Mandatory=$false)]
    [int]$DaysThreshold = 90,

    [Parameter(Mandatory=$false)]
    [string]$ArchivePath = "\\storageaccount.file.core.windows.net\fslogix-archive",

    [switch]$WhatIf

)

# ==========================================
# Function : Check if file is locked
# ==========================================

function Test-FileLocked {

    param(
        [string]$Path
    )

    try {

        $fs = [System.IO.File]::Open(
            $Path,
            [System.IO.FileMode]::Open,
            [System.IO.FileAccess]::ReadWrite,
            [System.IO.FileShare]::None
        )

        $fs.Close()

        return $false
    }

    catch {

        return $true
    }
}

# ==========================================
# Create Archive Folder if not exists
# ==========================================

if (-not (Test-Path $ArchivePath)) {

    if (-not $WhatIf) {

        New-Item `
            -Path $ArchivePath `
            -ItemType Directory `
            -Force | Out-Null
    }
}

# ==========================================
# Variables
# ==========================================

$Cutoff = (Get-Date).AddDays(-$DaysThreshold)

$LogFile = Join-Path `
    -Path $ArchivePath `
    -ChildPath ("FSLogixCleanupLog_{0:yyyyMMdd_HHmmss}.csv" -f (Get-Date))

$Results = @()

Write-Output "========================================="
Write-Output "FSLogix Cleanup Started"
Write-Output "Cutoff Date : $Cutoff"
Write-Output "========================================="

# ==========================================
# Scan VHD/VHDX Files
# ==========================================

Get-ChildItem `
    -Path $ProfileShare `
    -Include *.vhd, *.vhdx `
    -Recurse `
    -ErrorAction SilentlyContinue |

Where-Object {

    $_.LastWriteTime -lt $Cutoff

} |

ForEach-Object {

    $File = $_

    Write-Output "Processing : $($File.FullName)"

    # ==========================================
    # Result Object
    # ==========================================

    $Entry = [PSCustomObject]@{

        TimeChecked = Get-Date
        FileName    = $File.Name
        FilePath    = $File.FullName
        SizeGB      = [math]::Round($File.Length / 1GB, 2)
        LastWrite   = $File.LastWriteTime
        Action      = ""
    }

    # ==========================================
    # Check Locked File
    # ==========================================

    $IsLocked = Test-FileLocked -Path $File.FullName

    if ($IsLocked) {

        $Entry.Action = "SKIPPED - File Locked"

        $Results += $Entry

        Write-Output "Skipped (Locked) : $($File.Name)"

        return
    }

    # ==========================================
    # Archive Destination
    # ==========================================

    $Destination = Join-Path `
        -Path $ArchivePath `
        -ChildPath $File.Name

    try {

        if ($WhatIf) {

            $Entry.Action = "WHATIF - MOVE"

            Write-Output "WhatIf Move : $($File.Name)"
        }

        else {

            Move-Item `
                -Path $File.FullName `
                -Destination $Destination `
                -Force

            $Entry.Action = "MOVED"

            Write-Output "Moved : $($File.Name)"
        }
    }

    catch {

        $Entry.Action = "ERROR : $($_.Exception.Message)"

        Write-Output "Error : $($File.Name)"
    }

    $Results += $Entry
}

# ==========================================
# Export CSV Log
# ==========================================

if ($Results.Count -gt 0) {

    if (-not $WhatIf) {

        $Results | Export-Csv `
            -Path $LogFile `
            -NoTypeInformation `
            -Encoding UTF8

        Write-Output "========================================="
        Write-Output "Log Exported : $LogFile"
        Write-Output "========================================="
    }
}

Write-Output "FSLogix Cleanup Completed"

# ==========================================
# Example Commands
# ==========================================

<#

# Dry Run (Recommended First)

.\FSLogixCleanup.ps1 -WhatIf


# Actual Run

.\FSLogixCleanup.ps1


# Change Threshold to 120 Days

.\FSLogixCleanup.ps1 -DaysThreshold 120


# Custom FSLogix Share

.\FSLogixCleanup.ps1 `
-ProfileShare "\\storageaccount.file.core.windows.net\fslogix"


# Custom Archive Path

.\FSLogixCleanup.ps1 `
-ArchivePath "\\storageaccount.file.core.windows.net\fslogix-archive"

#>