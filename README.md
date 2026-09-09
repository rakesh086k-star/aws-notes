let VMStatus =
arg("").Resources
| where type =~ "microsoft.compute/virtualmachines"
| where resourceGroup =~ "RG-AVD-PH-EI-US"
| extend
    ComputerName = tostring(name),
    PowerStateCode = tostring(properties.extended.instanceView.powerState.code)
| extend
    Status = case(
        PowerStateCode == "PowerState/running", "Online",
        PowerStateCode == "PowerState/stopped", "Stopped",
        PowerStateCode == "PowerState/deallocated", "Deallocated",
        "Unavailable"
    )
| project
    VMId = tolower(tostring(id)),
    ComputerName,
    Status;

let CurrentUsers =
WVDConnections
| where TimeGenerated >= ago(24h)
| summarize arg_max(TimeGenerated, *) by CorrelationId
| where State == "Connected"
| extend
    VMId = tolower(tostring(SessionHostAzureVmId)),
    ComputerName = tostring(split(SessionHostName, ".")[0]),
    UserName = tostring(UserName)
| project
    VMId,
    ComputerName,
    UserName;

VMStatus
| join kind=leftouter CurrentUsers on VMId
| project
    ComputerName,
    UserName = iff(isempty(UserName), "No User", UserName),
    Status
| order by case(
    Status == "Online", 1,
    Status == "Stopped", 2,
    Status == "Deallocated", 3,
    Status == "Unavailable", 4,
    5
) asc,
ComputerName asc