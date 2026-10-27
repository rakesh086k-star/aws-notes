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
| summarize arg_max(TimeGenerated, *) by UserName, SessionHostName
| project
    UserName = tostring(UserName),
    ComputerName = tostring(SessionHostName),
    Status,
    TimeGenerated
| order by Status asc