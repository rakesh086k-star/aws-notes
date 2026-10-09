Import-Module ActiveDirectory

$UserID = "aalmarazher rera_wo"

Get-ADDomainController -Filter * | ForEach-Object {
    $DC = $_.HostName

    try {
        $User = Get-ADUser -Identity $UserID `
            -Server $DC -Properties lastLogon -ErrorAction Stop

        [PSCustomObject]@{
            UserID      = $UserID
            DomainController = $DC
            LastLogon   = if ($User.lastLogon -gt 0) {
                [DateTime]::FromFileTime($User.lastLogon)
            } else {
                "Never on this DC"
            }
        }
    }
    catch {
        [PSCustomObject]@{
            UserID      = $UserID
            DomainController = $DC
            LastLogon   = "Lookup failed"
        }
    }
} | Format-Table -AutoSize