# Input and output files
$InputFile  = "C:\Temp\UserList.csv"
$OutputFile = "C:\Temp\UserLastLogonReport.csv"
# Check Active Directory module
if (-not (Get-Module -ListAvailable -Name ActiveDirectory)) {
    Write-Host "ERROR: ActiveDirectory PowerShell module is not installed." -ForegroundColor Red
    Write-Host "Run this script on a machine with RSAT Active Directory tools."
    return
}
Import-Module ActiveDirectory
# Check input file
if (-not (Test-Path $InputFile)) {
    Write-Host "ERROR: Input file not found: $InputFile" -ForegroundColor Red
    return
}
# Read user IDs from CSV
$Users = Import-Csv -Path $InputFile
if (-not ($Users | Get-Member -Name User -MemberType NoteProperty)) {
    Write-Host "ERROR: CSV must contain a column named User." -ForegroundColor Red
    return
}
$Results = foreach ($Entry in $Users) {
    $UserID = ([string]$Entry.User).Trim()
    if ([string]::IsNullOrWhiteSpace($UserID)) {
        continue
    }
    Write-Host "Checking: $UserID"
    try {
        $ADUser = Get-ADUser -Identity $UserID `
            -Properties LastLogonDate, Enabled, WhenCreated `
            -ErrorAction Stop
        $LastLogon = if ($ADUser.LastLogonDate) {
            $ADUser.LastLogonDate
        } else {
            "Never / Not recorded"
        }
        [PSCustomObject]@{
            UserID       = $UserID
            DisplayName  = $ADUser.Name
            Enabled      = $ADUser.Enabled
            LastLogon    = $LastLogon
            AccountCreated = $ADUser.WhenCreated
            Status       = "Found"
        }
    }
    catch {
        [PSCustomObject]@{
            UserID       = $UserID
            DisplayName  = ""
            Enabled      = ""
            LastLogon    = "Unknown"
            AccountCreated = ""
            Status       = "Not found / Lookup failed"
        }
    }
}
# Export results
$Results | Export-Csv -Path $OutputFile `
    -NoTypeInformation -Encoding UTF8
Write-Host ""
Write-Host "Report generated successfully:" -ForegroundColor Green
Write-Host $OutputFile
# Open report in Excel if associated with CSV
Invoke-Item $OutputFile