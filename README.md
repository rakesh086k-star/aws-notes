WVDConnections
| where TimeGenerated > ago(24h)
| where State == "Failed"
| extend SourceClient = tostring(ClientType)
| extend ErrorMessageText = "Error details not available in this workspace"
| project TimeGenerated,
          UserName,
          SessionHostName,
          SourceClient,
          ErrorMessageText
| order by TimeGenerated desc