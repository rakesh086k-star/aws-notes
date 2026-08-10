Resources
| where type =~ "microsoft.compute/virtualmachines"
| extend
    VMName = name,
    VMSize = tostring(properties.hardwareProfile.vmSize),
    ComputerName = tostring(properties.osProfile.computerName),
    Location = location
| project
    VMName,
    ComputerName,
    VMSize,
    Location,
    ResourceGroup = resourceGroup,
    Subscription = subscriptionId
| order by VMSize asc, VMName asc