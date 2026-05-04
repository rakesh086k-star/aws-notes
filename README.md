WVDConnections
| where TimeGenerated > ago(24h)
| where State == "Failed"
| where isnotempty(ClientType)
| extend SourceClient = ClientType
| extend ErrorMessageText = coalesce(ErrorMessage, "No error message available")
| extend Highlight = "🔥 Connection Error"
| project TimeGenerated,
          UserName,
          SessionHostName,
          SourceClient,
          ErrorMessageText,
          Highlight
| order by TimeGenerated desc