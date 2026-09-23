arg("").Resources
| where type =~ "microsoft.compute/virtualmachines"
| where resourceGroup =~ "RG-AVD-PH-EI-US"
| extend
    ComputerName = tolower(tostring(name)),
    PowerState = tostring(properties.extended.instanceView.powerState.code)
| project ComputerName, PowerState