WVDSessions
| where TimeGenerated > ago(4h)
| summarize arg_max(TimeGenerated, *) by SessionId, UserName
| extend Status = case(
    SessionState =~ "Active", "Active",
    SessionState =~ "Connected", "Active",
    SessionState =~ "Disconnected", "Disconnected",
    SessionState =~ "LogOff", "Logged Off",
    SessionState =~ "Pending", "Pending",
    tostring(SessionState)
)
| project
    Status,
    ComputerName = tostring(SessionHostName),
    UserName = tostring(UserName),
    TimeGenerated
| order by Status asc, ComputerName asc, UserName as