WVDCheckpoints
| where TimeGenerated > ago(1h)
| where Name contains "Disconnected"
| summarize DisconnectedUsers = dcount(UserName)