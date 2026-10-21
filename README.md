$pkg = Get-AppxPackage MSTeams
Add-AppxPackage -DisableDevelopmentMode -Register "$($pkg.InstallLocation)\AppxManifest.xml"