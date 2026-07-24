print Metric="Idle Sessions",
Count=
toscalar(
WVDConnections
| where TimeGenerated > ago(1h)
| where State =~ "Idle"
| summarize dcount(CorrelationId)
)

print Metric="Idle Sessions",
Count=
toscalar(
WVDConnections
| where TimeGenerated > ago(1h)
| where ActivityType contains "Idle"
| summarize dcount(CorrelationId)
)