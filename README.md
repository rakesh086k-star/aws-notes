let Sessions =
    WVDConnections
    | summarize arg_max(TimeGenerated, *) by UserName;

let SessionCounts =
    Sessions
    | summarize
        ActiveSessions = countif(State == "Connected"),
        DisconnectedSessions = countif(State == "Disconnected");

let Hosts =
    Heartbeat
    | summarize arg_max(TimeGenerated, *) by Computer
    | summarize TotalSessionHosts = count();

SessionCounts
| extend TotalSessionHosts = toscalar(Hosts)
