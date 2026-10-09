$InputFile  = "C:\Temp\UserList.csv"
$OutputFile = "C:\Temp\UserLastLogonReport.csv"
if (!(Test-Path $InputFile)) {
    Write-Host "Input file not found: $InputFile" -ForegroundColor Red
    return
}
$Rows = Import-Csv -Path $InputFile
if (!$Rows -or $Rows.Count -eq 0) {
    Write-Host "CSV is empty or could not be read." -ForegroundColor Red
    return
}
$ColumnName = $Rows[0].PSObject.Properties.Name | Select-Object -First 1
$Results = foreach ($row in $Rows) {
    $id = ([string]$row.$ColumnName).Trim()
    if ([string]::IsNullOrWhiteSpace($id)) {
        continue
    }
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
$Results | Export-Csv -Path $OutputFile -NoTypeInformation -Encoding UTF8
Write-Host "Report saved to $OutputFile" -ForegroundColor Green
Invoke-Item $OutputFile