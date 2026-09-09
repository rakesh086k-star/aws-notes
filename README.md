Resources
| where type =~ "microsoft.compute/virtualmachines"
| where resourceGroup =~ "RG-AVD-PH-EI-US"
| extend
    ComputerName = tostring(name),
    PowerState = tostring(properties.extended.instanceView.powerState.code)
| extend
    Status = case(
        PowerState =~ "PowerState/running", "Online",
        PowerState =~ "PowerState/stopped", "Stopped",
        PowerState =~ "PowerState/deallocated", "Deallocated",
        "Unavailable"
    )
| summarize
    Online = countif(Status == "Online"),
    Stopped = countif(Status == "Stopped"),
    Deallocated = countif(Status == "Deallocated"),
    Unavailable = countif(Status == "Unavailable")