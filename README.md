// ==========================================================
// AVD VM Status + Computer Name + Current User Name
// Run in: Log Analytics Workspace -> Logs
// ==========================================================

let VMStatus =
    arg("").Resources
    | where type =~ "microsoft.compute/virtualmachines"
    | where resourceGroup =~ "RG-AVD-PH-EI-US"
    | extend
        ComputerName = tolower(tostring(name)),
        PowerState = tostring(properties.extended.instanceView.powerState.code)
    | extend
        Status = case(
            PowerState =~ "PowerState/running", "Online",
            PowerState =~ "PowerState/stopped", "Stopped",
            PowerState =~ "PowerState/deallocated", "Deallocated",
            "Unavailable"
        )
    | project ComputerName, Status;


// ==========================================================
// Get Current Connected AVD Users
// ==========================================================

let CurrentUsers =
    WVDConnections
    | where TimeGenerated >= ago(24h)
    | where State =~ "Connected"
    | extend
        ComputerName = tolower(tostring(split(SessionHostName, ".")[0])),
        UserName = tostring(UserName)
    | summarize
        LastSeen = max(TimeGenerated),
        Users = make_set(UserName)
        by ComputerName;


// ==========================================================
// Combine VM Status + User Information
// ==========================================================

VMStatus
| join kind=leftouter CurrentUsers on ComputerName
| extend
    UserName = case(
        isempty(Users),
        "No User",
        array_length(Users) == 1,
        tostring(Users[0]),
        strcat_array(Users, ", ")
    )
| project
    ComputerName,
    UserName,
    Status
| order by ComputerName asc