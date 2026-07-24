WVDCheckpoints
| where TimeGenerated > ago(1h)
| where Name contains "Disconnected"
| summarize DisconnectedUsers = dcount(UserName)


print Metric="Idle Sessions",
Count=
toscalar(
WVDConnections
| where TimeGenerated > ago(1h)
| where SessionState =~ "Idle"
| summarize dcount(CorrelationId)
)