WVDConnections
| summarize 
    ActiveUsers = dcountif(UserName, State == "Connected"),
    DisconnectedUsers = dcountif(UserName, State == "Disconnected"),
    ActiveSessions = countif(State == "Connected")




======

let base = WVDConnections
| summarize LastActivity = max(TimeGenerated) by UserName, SessionHost, State;

let connected = base
| where State == "Connected"
| extend IdleMinutes = datetime_diff("minute", now(), LastActivity);

let active = connected
| summarize ActiveUsers = dcountif(UserName, IdleMinutes <= 10),
            ActiveSessions = countif(IdleMinutes <= 10);

let idle = connected
| summarize IdleUsers = dcountif(UserName, IdleMinutes > 10),
            IdleSessions = countif(IdleMinutes > 10);

let disconnected = base
| where State == "Disconnected"
| summarize DisconnectedUsers = dcount(UserName),
            DisconnectedSessions = count();

active
| join kind=fullouter idle on 1==1
| join kind=fullouter disconnected on 1==1


ppppppppppppppppp

let IdleThreshold = 15;   // minutes (you can change)
let LatestConnections =
    WVDConnections
    | summarize arg_max(TimeGenerated, *) by UserName;

LatestConnections
| extend IdleMinutes = datetime_diff("minute", now(), TimeGenerated)
| summarize 
    ActiveSessions = countif(State == "Connected"),
    DisconnectedUsers = countif(State == "Disconnected"),
    IdleUsers = countif(State == "Connected" and IdleMinutes > IdleThreshold)
