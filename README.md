let locks = resources
| where type =~ "microsoft.authorization/locks"
| extend resourceGroup = resourceGroup
| extend lockName = name
| extend lockType = tostring(properties.level)
| extend notes = tostring(properties.notes)
| project resourceGroup, lockName, lockType, notes;

resourcecontainers
| where type == "microsoft.resources/subscriptions/resourcegroups"
| project resourceGroup = name, subscriptionId
| join kind=leftouter locks on resourceGroup
| extend lockStatus = iff(isnotempty(lockName), "Locked", "Not Locked")
| project subscriptionId, resourceGroup, lockStatus, lockName, lockType, notes
| order by resourceGroup asc
