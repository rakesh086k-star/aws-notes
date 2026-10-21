$exe = "$((Get-AppxPackage MSTeams).InstallLocation)\ms-teams.exe"
Get-Acl $exe | Format-List Owner, Access


Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-AppXDeploymentServer/Operational'
    StartTime = (Get-Date).AddHours(-4)
} -ErrorAction SilentlyContinue |
Where-Object { $_.Message -match 'MSTeams|ms-teams' } |
Select-Object -First 10 TimeCreated, Id, LevelDisplayName, Message |
Format-List