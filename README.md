WVDConnections
| where TimeGenerated > ago(1h)
| summarize arg_max(TimeGenerated, *) by CorrelationId
| summarize
    ActiveSessions = countif(State =~ "Connected"),
    DisconnectedSessions = countif(State =~ "Disconnected"),
    TotalSessions = count()
