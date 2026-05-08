WVDConnections
| where TimeGenerated > ago(30d)
| where State in ("ConnectionFailed", "DisconnectedByUser")
| summarize Issues=count() by SessionHostName, State
| order by Issues desc