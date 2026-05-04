WVDConnections
| where TimeGenerated > ago(24h)
| extend IsError = iff(State == "Failed", "Error", "OK")
| extend IsClient = iff(ClientType != "", "Client", "Other")
| extend Highlight = case(
    State == "Failed" and ClientType != "", "🔥 Client Error",
    State == "Failed", "❗ Error",
    ClientType != "", "🔹 Client",
    "Normal"
)
| project TimeGenerated, UserName, ClientType, State, Highlight
| order by TimeGenerated des