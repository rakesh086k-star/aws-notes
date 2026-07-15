Get-AppxPackage -AllUsers *Handwriting* | Remove-AppxPackage



Get-AppxProvisionedPackage -Online |
Where-Object DisplayName -like "*Handwriting*" |
Remove-AppxProvisionedPackage -Online


Resources
| where type =~ "microsoft.compute/virtualMachines"
| where resourceGroup == "RG-AVD-WRR-E1-US"
| summarize Count=count() by PowerState=tostring(properties.extended.instanceView.powerState.code)

