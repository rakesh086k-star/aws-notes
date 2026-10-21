$pkg = Get-AppxPackage MSTeams
Test-Path "$($pkg.InstallLocation)\ms-teams.exe"


Get-AppxPackage MSTeams |
Format-List Name, Version, Status, InstallLocation, PackageUserInformation


explorer.exe "shell:AppsFolder\MSTeams_8wekyb3d8bbwe!MSTeams"