// ==========================================
// AVD User Session Status - Active / Disconnected / Idle
// Shows: Metric, Computer Name, User Name
// ==========================================

print Metric = "Active Users"
| extend ComputerName = "", UserName = ""
| union (
    WVDConnections
    | where TimeGenerated > ago(4h)
    | where isnotempty(UserName)
    | extend Metric = "Active Users"
    | extend ComputerName = tostring(SessionHostName)
    | extend UserName = tostring(UserName)
    | project Metric, ComputerName, UserName
)
| union (
    WVDCheckpoints
    | where TimeGenerated > ago(1h)
    | where Name contains "Disconnected"
    | where isnotempty(UserName)
    | extend Metric = "Disconnected Sessions"
    | extend ComputerName = tostring(SessionHostName)
    | extend UserName = tostring(UserName)
    | project Metric, ComputerName, UserName
)
| union (
    WVDAgentHealthStatus
    | where TimeGenerated > ago(1h)
    | extend Metric = "Idle Sessions"
    | extend ComputerName = tostring(SessionHostName)
    | extend UserName = "N/A"
    | project Metric, ComputerName, UserName
)
| order by Metric asc, ComputerName asc, UserName asc