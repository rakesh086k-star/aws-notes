WVDConnections
| where TimeGenerated >= ago(24h)
| where State in ("Connected", "Completed")
| extend
    ComputerName = tostring(split(SessionHostName, ".")[0]),
    UserName = tostring(UserName)
| summarize
    arg_max(TimeGenerated, State)
    by CorrelationId, ComputerName, UserName
| extend
    SessionStatus = case(
        State == "Connected", "Online",
        State == "Completed", "Disconnected",
        "Unknown"
    )
| project
    ComputerName,
    UserName,
    SessionStatus,
    LastSeen = TimeGenerated
| order by SessionStatus asc, ComputerName asc