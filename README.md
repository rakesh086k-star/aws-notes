resources
| where type == "microsoft.authorization/locks"
| extend lockName = name
| extend lockType = properties.level
| extend notes = properties.notes
| extend scope = tostring(properties.scope)
| extend resourceGroup = resourceGroup
| project subscriptionId, resourceGroup, lockName, lockType, scope, notes
| order by resourceGroup asc

$result | Export-Csv "C:\Temp\RG-Lock-Report.csv" -NoTypeInformation
