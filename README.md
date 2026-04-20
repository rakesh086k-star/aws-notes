resourcecontainers
| where type == "microsoft.resources/subscriptions/resourcegroups"
| extend rgId = tolower(id)
| project resourceGroup = name, subscriptionId, rgId
| join kind=leftouter (
    resources
    | where type =~ "microsoft.authorization/locks"
    | extend lockScope = tolower(id)
    | extend lockName = name
    | extend lockType = tostring(properties.level)
    | extend notes = tostring(properties.notes)
    | project lockScope, lockName, lockType, notes
) on $left.rgId == $right.lockScope
| extend lockStatus = iff(isnotempty(lockName), "Locked", "Not Locked")
| extend lockName = coalesce(lockName, "No Lock")
| extend lockType = coalesce(lockType, "None")
| extend notes = coalesce(notes, "")
| project subscriptionId, resourceGroup, lockStatus, lockName, lockType, notes
| order by resourceGroup asc
