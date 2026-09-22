Resources
| where type =~ "microsoft.desktopvirtualization/hostpools"
| project
    subscriptionId,
    resourceGroup,
    name,
    type,
    location,
    id
| order by subscriptionId, name
