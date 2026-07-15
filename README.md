Get-AppxPackage -AllUsers *Handwriting* | Remove-AppxPackage



Get-AppxProvisionedPackage -Online |
Where-Object DisplayName -like "*Handwriting*" |
Remove-AppxProvisionedPackage -Online



search *
| summarize Count=count() by $table
| sort by $table asc


union withsource=TableName *
| summarize Count=count() by TableName
| sort by TableName asc


Resources
| where type =~ "microsoft.compute/virtualMachines"
| where resourceGroup == "RG-AVD-WRR-E1-US"
| summarize Count=count() by PowerState=tostring(properties.extended.instanceView.powerState.code)


WVDConnections
| where TimeGenerated >= ago(24h)
| summarize Sessions = count() by GatewayRegion



WVDConnections
| where TimeGenerated >= ago(24h)
| extend Country = geo_info_from_ip_address(ClientIPAddress).country
| summarize Users = dcount(UserName) by Country
| order by Users desc


