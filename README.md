// =====================================================
// AVD VM Power State + Current Connected User
// Run in: Log Analytics Workspace -> Logs
// =====================================================

let VMStatus =
    arg("").Resources
    | where type =~ "microsoft.compute/virtualmachines"
    | where resourceGroup =~ "RG-AVD-PH-EI-US"
    | extend
        ComputerName = tolower(tostring(name)),
        PowerStateCode = tostring(properties.extended.instanceView.powerState.code)
    | extend
        Status = case(
            PowerStateCode =~ "PowerState/running", "Online",
            PowerStateCode =~ "PowerState/stopped", "Stopped",
            PowerStateCode =~ "PowerState/deallocated", "Deallocated",
            "Unavailable"
        )
    | project ComputerName, Status;

// =====================================================
// Current Connected AVD Users
// =====================================================

let CurrentUsers =
    WVDConnections
    | where TimeGenerated >= ago(24h)
    | where State =~ "Connected"
    | extend
        ComputerName = tolower(
            tostring(split(SessionHostName, ".")[0])
        )
    | summarize
        LastSeen = max(TimeGenerated),
        UserNames = make_set(UserName)
        by ComputerName;

// =====================================================
// Combine VM Status + User Information
// =====================================================

VMStatus
| join kind=leftouter hint.remote=right CurrentUsers
    on ComputerName
| extend
    UserName = case(
        isempty(tostring(UserNames)), "No User",
        tostring(UserNames)
    )
| project
    ComputerName,
    UserName,
    Status
| order by ComputerName asc