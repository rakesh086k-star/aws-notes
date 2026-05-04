Heartbeat
| summarize LastSeen = max(TimeGenerated) by Computer
| extend Status = iff(LastSeen > ago(10m), "Active", "Inactive")
| summarize Count = count() by Status