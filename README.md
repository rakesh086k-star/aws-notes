let ActiveUsers =
toscalar(
    WVDConnections
    | where TimeGenerated > ago(24h)
    | summarize dcount(UserName)
);

let DisconnectedSessions =
toscalar(
    WVDCheckpoints
    | where TimeGenerated > ago(24h)
    | where Name contains "Disconnected"
    | summarize count()
);

let IdleSessions =
toscalar(
    WVDAgentHealthStatus
    | where TimeGenerated > ago(15m)
    | summarize dcount(SessionHostName)
);

print
    ActiveUsers = ActiveUsers,
    DisconnectedSessions = DisconnectedSessions,
    IdleSessions = IdleSessions