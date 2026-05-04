WVDConnections
| where TimeGenerated > ago(24h)
| extend SourceClient = tostring(ClientType)
| extend Status = case(
    State == "Active", "🟢 Active",
    State == "Connected", "🟢 Connected",
    State == "Disconnected", "🟡 Disconnected",
    State == "Failed", "🔴 Connection Error",
    "⚪ Unknown"
)
| project TimeGenerated,
          UserName,
          SessionHostName,
          SourceClient,
          State,
          Status
| order by TimeGenerated desc