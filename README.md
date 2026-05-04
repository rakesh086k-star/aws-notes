WVDConnections
| summarize 
    ActiveUsers = dcountif(UserName, State == "Connected"),
    DisconnectedUsers = dcountif(UserName, State == "Disconnected"),
    ActiveSessions = countif(State == "Connected")
