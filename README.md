Start-Process "$env:LOCALAPPDATA\Microsoft\WindowsApps\ms-teams.exe"



Get-AppxPackage MSTeams | Select-Object -ExpandProperty InstallLocation