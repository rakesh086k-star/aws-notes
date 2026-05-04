WVDActiveSessions
| summarize
    ActiveSessions = countif(SessionState =~ "Active"),
    DisconnectedSessions = countif(SessionState =~ "Disconnected"),
    TotalSessions = count()
