Get-AppxPackage -AllUsers *Handwriting* | Remove-AppxPackage



Get-AppxProvisionedPackage -Online |
Where-Object DisplayName -like "*Handwriting*" |
Remove-AppxProvisionedPackage -Online


