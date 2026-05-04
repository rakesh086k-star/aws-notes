WVDConnections
| where TimeGenerated > ago(24h)
| where State == "Failed"
| extend SourceClient = tostring(ClientType)
| extend ErrorMessageText = "No detailed error available in WVDConnections table"
| project TimeGenerated,
          UserName,
          SessionHostName,
          SourceClient,
          ErrorMessageText
| order by TimeGenerated desc