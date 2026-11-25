Add-AppxProvisionedPackage -Online `
  -PackagePath "C:\Temp\Slack.msix" `
  -SkipLicense `
  -Regions "all"



Get-AppxProvisionedPackage -Online |
Where-Object {$_.DisplayName -like "*Slack*"}