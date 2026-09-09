let VMStatus =
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
    | project ComputerName, VMStatus;

let ConnectedUsers =
    WVDConnections
    | where TimeGenerated >= ago(24h)
    | where State =~ "Connected"
    | extend
        ComputerName = tolower(tostring(split(SessionHostName, ".")[0])),
        UserName = tostring(UserName)
    | summarize UserNames = make_set(UserName) by ComputerName;

VMStatus
| join kind=leftouter hint.remote=right ConnectedUsers on ComputerName
| extend
    UserName = iff(
        isempty(UserNames),
        "No User",
        strcat_array(UserNames, ", ")
    )
| project
    ComputerName,
    UserName,
    VMStatus
| order by ComputerName asc