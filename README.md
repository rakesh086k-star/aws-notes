$InputFile = "C:\Temp\UserList.csv"
$OutputFile = "C:\Temp\UserLastLogonReport.csv"
New-Item -ItemType Directory -Path "C:\Temp" -Force | Out-Null
$Results = foreach ($row in (Import-Csv $InputFile)) {
    $id = $row.UserID.Trim()
    if (!$id) { continue }
    Write-Host "Checking $id ..."
    $output = net.exe user $id /domain 2>&1 | Out-String
    $match = [regex]::Match(
        $output,
        '(?im)^\s*Last logon\s+(.+?)\s*$'
    )
    if ($match.Success) {
        $lastLogon = $match.Groups[1].Value.Trim()
        $status = "OK"
    }
    else {
        $lastLogon = "Could not read"
        $status = "Check command output"
    }
    [PSCustomObject]@{
        UserID    = $id
        LastLogon = $lastLogon
        Status    = $status
    }
}
$Results | Export-Csv $OutputFile -NoTypeInformation -Encoding UTF8
Write-Host "Report saved to $OutputFile" -ForegroundColor Green
Invoke-Item $OutputFile