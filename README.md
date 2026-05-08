WVDConnections
| where TimeGenerated > ago(30d)
| summarize Count=count() by State
| order by Count desc