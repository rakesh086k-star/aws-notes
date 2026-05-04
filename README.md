WVDConnections
| summarize arg_max(TimeGenerated, *) by SessionId
| summarize
    ActiveSessions = countif(State == "Connected"),
    DisconnectedSessions = countif(State == "Disconnected"),
    TotalSessions = count()
