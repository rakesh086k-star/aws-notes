resourcecontainers
| where type == "microsoft.resources/subscriptions/resourcegroups"
| project resourceGroup = name, subscriptionId
| join kind=leftouter (
    resources
    | where type =~ "microsoft.authorization/locks"
    | project resourceGroup, lockName = name, lockType = tostring(properties.level), notes = tostring(properties.notes)
) on resourceGroup
| extend lockStatus = iff(isnotempty(lockName), "Locked", "Not Locked")
| project subscriptionId, resourceGroup, lockStatus, lockName, lockType, notes
