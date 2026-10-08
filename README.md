Import-Module ActiveDirectory

$InputFile = "C:\UserList.csv"
$OutputFile = "C:\User_Status_Report.csv"

$Results = foreach ($User in Import-Csv $InputFile) {

    $UserID = $User.UserID.Trim()

    try {
        $ADUser = Get-ADUser -Identity $UserID -Properties Enabled, LockedOut, LastLogonDate

        if ($ADUser.Enabled -eq $true) {
            $Status = "Active"
        }
        else {
            $Status = "Disabled"
        }

        [PSCustomObject]@{
            UserID       = $UserID
            Status       = $Status
            LockedOut    = $ADUser.LockedOut
            LastLogon    = $ADUser.LastLogonDate
        }
    }
    catch {
        [PSCustomObject]@{
            UserID       = $UserID
            Status       = "Not Found"
            LockedOut    = ""
            LastLogon    = ""
        }
    }
}

$Results | Export-Csv $OutputFile -NoTypeInformation

Write-Host "Completed!"
Write-Host "Report saved at: $OutputFile"