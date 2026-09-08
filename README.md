WVDConnections
| where TimeGenerated > ago(4h)
| where isnotempty(UserName)
| summarize arg_max(TimeGenerated, *) by CorrelationId
| extend Status = case(
    State == "Connected", "Active",
    State == "Started", "Active",
    State == "Completed", "Disconnected",
    State
)
| project
    Status,
    ComputerName = tostring(SessionHostName),
    UserName = tostring(UserName),
    TimeGenerated
| order by Status asc, ComputerName asc, UserName asc