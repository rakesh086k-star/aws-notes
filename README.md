let IdleThreshold = 30m;

let LatestSessions =
    WVDConnections
    | summarize arg_max(TimeGenerated, *) by UserName;

LatestSessions
| extend TimeDiff = now() - TimeGenerated
| summarize
    ActiveSessions = countif(State == "Connected"),
    DisconnectedUsers = countif(State == "Disconnected"),
    IdleUsers = countif(State == "Disconnected" and TimeDiff > IdleThreshold)
