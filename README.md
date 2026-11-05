Get-AppxProvisionedPackage -Online |
Where-Object {$_.DisplayName -like "*OutlookForWindows*"}



Get-AppxPackage -AllUsers Microsoft.OutlookForWindows