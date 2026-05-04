WVDConnections
| where TimeGenerated > ago(24h)
| extend SourceClient = tostring(ClientType)
| extend IsError = iff(State == "Failed", 1, 0)
| extend ErrorStatus = case(
    State == "Failed", "🔴 Connection Error (WVD)",
    State == "Disconnected", "🟡 Session Disconnected",
    State == "Active", "🟢 Healthy Session",
    "⚪ Unknown"
)
| extend CheckPoint = strcat("SessionCheck_", State)
| project TimeGenerated,
          UserName,
          SessionHostName,
          SourceClient,
          State,
          IsError,
          ErrorStatus,
          CheckPoint
| order by TimeGenerated desc