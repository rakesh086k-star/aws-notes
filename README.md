let StartTime = datetime(2026-09-17 00:00:00);
let EndTime = datetime(2026-09-17 23:59:59);
let TargetUser = "sritchie_wo@exlservice.com";

WVDErrors
| where TimeGenerated between (StartTime .. EndTime)
| where UserName =~ TargetUser
| project TimeGenerated, UserName, CodeSymbolic, Message, CorrelationId
| order by TimeGenerated asc





let StartTime = datetime(2026-09-17 12:20:00);
let EndTime   = datetime(2026-09-17 13:00:00);
let TargetUser = "sritchie_wo@exlservice.com";

WVDErrors
| where TimeGenerated between (StartTime .. EndTime)
| where UserName =~ TargetUser
| project
    TimeGenerated,
    CodeSymbolic,
    Message,
    CorrelationId,
    SessionHostName
| order by TimeGenerated asc





# ==========================================
# FSLogix Health Check - Read Only
# ==========================================

$Results = @()

function Add-Check {
    param(
        [string]$Category,
        [string]$Check,
        [string]$Status,
        [string]$Finding
    )

    $Results += [PSCustomObject]@{
        Category = $Category
        Check    = $Check
        Status   = $Status
        Finding  = $Finding
    }
}

# ------------------------------------------
# 1. FSLogix Installation
# ------------------------------------------

$FSLogixPath = "HKLM:\SOFTWARE\FSLogix\Apps"

if (Test-Path $FSLogixPath) {

    $FSVersion = (Get-ItemProperty $FSLogixPath -ErrorAction SilentlyContinue).InstallVersion

    Add-Check "INSTALL" "FSLogix Installed" "PASS" `
        "FSLogix is installed. Version: $FSVersion"

}
else {

    Add-Check "INSTALL" "FSLogix Installed" "FAIL" `
        "FSLogix registry path not found."
}

# ------------------------------------------
# 2. FSLogix Services
# ------------------------------------------

$Service = Get-Service frxsvc -ErrorAction SilentlyContinue

if ($Service -and $Service.Status -eq "Running") {

    Add-Check "SERVICE" "FSLogix Service" "PASS" `
        "frxsvc is Running."

}
else {

    Add-Check "SERVICE" "FSLogix Service" "FAIL" `
        "frxsvc is not running."
}

# ------------------------------------------
# 3. Profile Container
# ------------------------------------------

$ProfilePath = "HKLM:\SOFTWARE\FSLogix\Profiles"

if (Test-Path $ProfilePath) {

    $Profile = Get-ItemProperty $ProfilePath

    if ($Profile.Enabled -eq 1) {

        Add-Check "CONFIGURATION" "Profile Container" "PASS" `
            "Profile Container is enabled."

    }
    else {

        Add-Check "CONFIGURATION" "Profile Container" "FAIL" `
            "Profile Container is disabled."
    }

}
else {

    Add-Check "CONFIGURATION" "Profile Container" "FAIL" `
        "FSLogix Profiles registry key not found."
}

# ------------------------------------------
# 4. VHDLocations
# ------------------------------------------

$VHDLocations = (Get-ItemProperty $ProfilePath -ErrorAction SilentlyContinue).VHDLocations

if ($VHDLocations) {

    Add-Check "STORAGE" "VHDLocations" "PASS" `
        "VHDLocations configured: $VHDLocations"

}
else {

    Add-Check "STORAGE" "VHDLocations" "FAIL" `
        "VHDLocations is not configured."
}

# ------------------------------------------
# 5. Volume Type
# ------------------------------------------

$VolumeType = (Get-ItemProperty $ProfilePath -ErrorAction SilentlyContinue).VolumeType

if ($VolumeType -eq "vhdx") {

    Add-Check "CONFIGURATION" "VolumeType" "PASS" `
        "VolumeType is VHDX."

}
else {

    Add-Check "CONFIGURATION" "VolumeType" "WARN" `
        "VolumeType is '$VolumeType'."
}

# ------------------------------------------
# 6. DeleteLocalProfileWhenVHDShouldApply
# ------------------------------------------

$DeleteLocal = (Get-ItemProperty $ProfilePath -ErrorAction SilentlyContinue).DeleteLocalProfileWhenVHDShouldApply

if ($DeleteLocal -eq 1) {

    Add-Check "CONFIGURATION" `
        "DeleteLocalProfileWhenVHDShouldApply" `
        "PASS" `
        "Configured as 1."

}
else {

    Add-Check "CONFIGURATION" `
        "DeleteLocalProfileWhenVHDShouldApply" `
        "WARN" `
        "Configured as $DeleteLocal."
}

# ------------------------------------------
# 7. FlipFlopProfileDirectoryName
# ------------------------------------------

$FlipFlop = (Get-ItemProperty $ProfilePath -ErrorAction SilentlyContinue).FlipFlopProfileDirectoryName

if ($FlipFlop -eq 1) {

    Add-Check "CONFIGURATION" `
        "FlipFlopProfileDirectoryName" `
        "PASS" `
        "Configured as 1."

}
else {

    Add-Check "CONFIGURATION" `
        "FlipFlopProfileDirectoryName" `
        "WARN" `
        "Configured as $FlipFlop."
}

# ------------------------------------------
# 8. LockedRetryCount
# ------------------------------------------

$RetryCount = (Get-ItemProperty $ProfilePath -ErrorAction SilentlyContinue).LockedRetryCount

if ($RetryCount -eq 3) {

    Add-Check "CONFIGURATION" `
        "LockedRetryCount" `
        "PASS" `
        "Configured as 3."

}
else {

    Add-Check "CONFIGURATION" `
        "LockedRetryCount" `
        "INFO" `
        "Configured as $RetryCount."
}

# ------------------------------------------
# 9. Defender Exclusions
# ------------------------------------------

try {

    $Defender = Get-MpPreference -ErrorAction Stop

    $ProcessExclusions = $Defender.ExclusionProcess

    $PathExclusions = $Defender.ExclusionPath

    $FSLogixExcluded = $false

    if ($ProcessExclusions -match "frxsvc.exe") {
        $FSLogixExcluded = $true
    }

    if ($ProcessExclusions -match "frxccds.exe") {
        $FSLogixExcluded = $true
    }

    if ($FSLogixExcluded) {

        Add-Check "SECURITY" `
            "Defender FSLogix Exclusions" `
            "PASS" `
            "FSLogix process exclusion detected."

    }
    else {

        Add-Check "SECURITY" `
            "Defender FSLogix Exclusions" `
            "WARN" `
            "FSLogix process exclusions were not detected."

    }

}
catch {

    Add-Check "SECURITY" `
        "Defender FSLogix Exclusions" `
        "INFO" `
        "Unable to query Microsoft Defender."
}

# ------------------------------------------
# 10. FSLogix Logs
# ------------------------------------------

$LogPath = "C:\ProgramData\FSLogix\Logs"

if (Test-Path $LogPath) {

    $RecentLog = Get-ChildItem $LogPath -Recurse -File |
        Sort-Object LastWriteTime -Descending |
        Select-Object -First 1

    Add-Check "LOGGING" `
        "FSLogix Logs" `
        "PASS" `
        "FSLogix logs available."

}
else {

    Add-Check "LOGGING" `
        "FSLogix Logs" `
        "WARN" `
        "FSLogix log directory not found."
}

# ------------------------------------------
# SUMMARY
# ------------------------------------------

Write-Host ""
Write-Host "============================================="
Write-Host "        FSLogix HEALTH CHECK"
Write-Host "============================================="
Write-Host ""

$Results | Format-Table -AutoSize

$Pass = ($Results | Where-Object Status -eq "PASS").Count
$Warn = ($Results | Where-Object Status -eq "WARN").Count
$Fail = ($Results | Where-Object Status -eq "FAIL").Count
$Info = ($Results | Where-Object Status -eq "INFO").Count
$Total = $Results.Count

# Simple score
$Score = if ($Total -gt 0) {
    [math]::Round((($Pass + ($Warn * 0.5)) / $Total) * 100,1)
}
else {
    0
}

Write-Host ""
Write-Host "PASS     : $Pass"
Write-Host "WARNING  : $Warn"
Write-Host "FAIL     : $Fail"
Write-Host "INFO     : $Info"
Write-Host "TOTAL    : $Total"
Write-Host ""
Write-Host "HEALTH SCORE : $Score / 100"
Write-Host ""

if ($Fail -gt 0) {
    Write-Host "STATUS : CRITICAL"
}
elseif ($Warn -gt 0) {
    Write-Host "STATUS : WARNING"
}
else {
    Write-Host "STATUS : HEALTHY"
}

Write-Host ""
