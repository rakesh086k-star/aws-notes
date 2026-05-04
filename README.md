WVDConnections
| where TimeGenerated > ago(24h)
| where State == "Failed"
| extend SourceClient = tostring(ClientType)
| extend ErrorMessageText = tostring(coalesce(ErrorDetails, FailureReason, "No error message available"))
| project TimeGenerated,
          UserName,
          SessionHostName,
          SourceClient,
          ErrorMessageText
| order by TimeGenerated desc