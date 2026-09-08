// ============================================
// AVD User Session Details
// Active / Disconnected / Idle
// ============================================

let ActiveUsers =
    WVDConnections
    | where TimeGenerated > ago(4h)
    | where State == "Connected"
    | where isnotempty(UserName)
    | summarize arg_max(TimeGenerated, *) by CorrelationId
    | extend Metric = "Active Users"
    | project
        Metric,
        ComputerName = tostring(SessionHostName),
        UserName = tostring(UserName),
        TimeGenerated;

let DisconnectedUsers =
    WVDCheckpoints
    | where TimeGenerated > ago(4h)
    | where Name contains "Disconnected"
    | where isnotempty(UserName)
    | project
        TimeGenerated,
        CorrelationId,
        UserName,
        Name
    | join kind=leftouter
    (
        WVDConnections
        | project
            CorrelationId,
            SessionHostName
    )
    on CorrelationId
    | extend Metric = "Disconnected Sessions"
    | project
        Metric,
        ComputerName = tostring(SessionHostName),
        UserName = tostring(UserName),
        TimeGenerated;

let IdleHosts =
    WVDAgentHealthStatus
    | where TimeGenerated > ago(1h)
    | where toint(InactiveSessions) > 0
    | summarize arg_max(TimeGenerated, *) by _ResourceId
    | extend Metric = "Idle Sessions"
    | project
        Metric,
        ComputerName = tostring(extract(@"[^/]+$", 0, _ResourceId)),
        UserName = "N/A",
        TimeGenerated;

union ActiveUsers, DisconnectedUsers, IdleHosts
| order by Metric asc, ComputerName asc, UserName asc