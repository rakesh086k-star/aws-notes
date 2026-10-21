Get-WinEvent -LogName "Microsoft-Windows-AppXDeploymentServer/Operational" -MaxEvents 50 |
Where-Object { $_.LevelDisplayName -eq "Error" } |
Select-Object TimeCreated, Id, Message |
Format-List