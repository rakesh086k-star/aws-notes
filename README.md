Resources
| where type =~ "microsoft.compute/virtualmachines"
| project
    VMName = name,
    ResourceGroup = resourceGroup,
    SubscriptionId = subscriptionId,
    Location = location
| limit 10