Get-AppxProvisionedPackage -Online |
Where-Object {$_.DisplayName -like "*Get-AppxPackage -AllUsers |
Where-Object {$_.Name -like "*Slack*"} |
Select-Object Name, PackageFullName


