arg("").Resources
| where type =~ "microsoft.compute/virtualmachines"
| where resourceGroup =~ "RG-AVD-PH-EI-US"
| extend
    ComputerName = tolower(tostring(name)),
    PowerState = tostring(properties.extended.instanceView.powerState.code)
| extend
    VMStatus = case(
        PowerState =~ "PowerState/running", "Online",
        PowerState =~ "PowerState/stopped", "Stopped",
        PowerState =~ "PowerState/deallocated", "Deallocated",
        "Unavailable"
    )
| project ComputerName, VMStatus
| join kind=leftouter hint.remote=right (
    WVDConnections
    | where TimeGenerated >= ago(24h)
    | extend
        ComputerName = tolower(tostring(split(SessionHostName, ".")[0])),
        UserName = tostring(UserName)
    | summarize arg_max(TimeGenerated, State) by ComputerName, UserName
    | where State =~ "Connected"
    | project ComputerName, UserName
) on ComputerName
| extend UserName = iff(
    isempty(UserName),
    "No User",
    UserName
)
| project
    ComputerName,
    UserName,
    VMStatus
| order by ComputerName asc, UserName asc