Get-AppxProvisionedPackage -Online |
Where-Object {$_.DisplayName -like "*Slack*"}