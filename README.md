WVDConnections
| extend Status = case(
    State == "Active", "Green - Active",
    State == "Connected", "Green - Connected",
    State == "Disconnected", "Yellow - Disconnected",
    State == "Failed", "Red - Connection Error",
    "Gray - Unknown"
)
| extend IsError = iff(State == "Failed", 1, 0)
| extend HasSession = iff(isnotempty(UserName), 1, 0)
| project TimeGenerated, UserName, SessionHostName, State, Status, IsError, HasSession
| order by TimeGenerated desc