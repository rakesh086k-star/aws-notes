WVDConnections
| where HostPoolName == "<Your-HostPool-Name>"
| summarize arg_max(TimeGenerated, State) by UserName, Computer
| summarize 
    LoggedIn = countif(State == "Connected"),
    Disconnected = countif(State == "Disconnected")
by Computer