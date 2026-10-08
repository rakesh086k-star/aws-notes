# ============================================================
# AD USER ACCOUNT STATUS CHECK
# ============================================================

Import-Module ActiveDirectory

# Input and Output files
$InputFile  = "C:\Temp\UserList.csv"
$OutputFile = "C:\Temp\UserAccountStatus.csv"

# Check input file
if (-not (Test-Path $InputFile)) {
    Write-Host "ERROR: Input file not found: $InputFile" -ForegroundColor Red
    exit
}

# Import users
$Users = Import-Csv -Path $InputFile

$Results = @()

foreach ($User in $Users) {

    # Safely read UserID and remove spaces
    $UserID = ([string]$User.UserID).Trim()

    # Skip blank rows
    if ([string]::IsNullOrWhiteSpace($UserID)) {
        continue
    }

    # Remove DOMAIN\ from UserID if present
    if ($UserID -match "\\") {
        $UserID = $UserID.Split("\")[-1]
    }

    # Remove @domain.com if UPN is provided
    if ($UserID -match "@") {
        $UserID = $UserID.Split("@")[0]
    }

    Write-Host "Checking: $UserID" -ForegroundColor Cyan

    try {

        # Get AD User
        $ADUser = Get-ADUser `
            -Identity $UserID `
            -Properties Enabled,LockedOut,LastLogonDate,DisplayName,UserPrincipalName `
            -ErrorAction Stop

        # Determine account status
        if ($ADUser.Enabled -eq $true) {
            $Status = "Enabled"
        }
        else {
            $Status = "Disabled"
        }

        # Locked status
        if ($ADUser.LockedOut -eq $true) {
            $LockedOut = "Yes"
        }
        else {
            $LockedOut = "No"
        }

        # Last logon
        if ($null -ne $ADUser.LastLogonDate) {
            $LastLogon = $ADUser.LastLogonDate.ToString("yyyy-MM-dd HH:mm:ss")
        }
        else {
            $LastLogon = "Never"
        }

        # Add result
        $Results += [PSCustomObject]@{
            UserID            = $UserID
            DisplayName       = $ADUser.DisplayName
            UserPrincipalName = $ADUser.UserPrincipalName
            Status            = $Status
            LockedOut         = $LockedOut
            LastLogon         = $LastLogon
        }

    }
    catch {

        # User not found / other error
        $Results += [PSCustomObject]@{
            UserID            = $UserID
            DisplayName       = ""
            UserPrincipalName = ""
            Status            = "Not Found"
            LockedOut         = ""
            LastLogon         = ""
        }

        Write-Host "User not found: $UserID" -ForegroundColor Yellow
    }
}

# Export results
$Results | Export-Csv `
    -Path $OutputFile `
    -NoTypeInformation `
    -Encoding UTF8

Write-Host ""
Write-Host "============================================" -ForegroundColor Green
Write-Host "Completed!" -ForegroundColor Green
Write-Host "============================================" -ForegroundColor Green
Write-Host "Report saved at:" -ForegroundColor Green
Write-Host $OutputFile -ForegroundColor White
Write-Host ""

# Display results on screen
$Results | Format-Table -AutoSize