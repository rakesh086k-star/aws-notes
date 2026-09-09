Resources
| where type =~ "microsoft.compute/virtualmachines"
| where resourceGroup =~ "RG-AVD-PH-EI-US"
| extend
    ComputerName = tostring(name),
    powerState = tostring(properties.extended.instanceView.powerState.code)
| summarize
    Online = countif(powerState == "PowerState/running"),
    Stopped = countif(powerState == "PowerState/stopped"),
    Deallocated = countif(powerState == "PowerState/deallocated"),
    Unavailable = countif(
        powerState != "PowerState/running"
        and powerState != "PowerState/stopped"
        and powerState != "PowerState/deallocated"
    )
| project
    Status = pack_array("Online", "Stopped", "Deallocated", "Unavailable"),
    Count = pack_array(Online, Stopped, Deallocated, Unavailable)
| mv-expand Status, Count
| project
    Status = tostring(Status),
    Count = toint(Count)
| order by case(
    Status == "Online", 1,
    Status == "Stopped", 2,
    Status == "Deallocated", 3,
    Status == "Unavailable", 4,
    5
) as