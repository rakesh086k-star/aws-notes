WVDConnections
| where TimeGenerated > ago(24h)   // important for performance + freshness
| summarize arg_max(TimeGenerated, State) by CorrelationId
| summarize
    ActiveSessions = countif(State has "Connect"),
    DisconnectedSessions = countif(State has "Disconnect"),
    TotalSessions = count()
