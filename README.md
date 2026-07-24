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




   



WVDConnections
| where TimeGenerated >= ago(90d)
| summarize LastLogin = max(TimeGenerated) by UserName
| extend DaysSinceLastLogin = datetime_diff("day", now(), LastLogin)
| extend LoginStatus = case(
    DaysSinceLastLogin == 0, "🟢 Today",
    DaysSinceLastLogin == 1, "🟡 Yesterday",
    DaysSinceLastLogin <= 7, "🟠 Last 7 Days",
    DaysSinceLastLogin <= 30, "🔵 Last 30 Days",
    "🔴 Inactive (30+ Days)"
)
| project
    UserName,
    LastLogin,
    DaysSinceLastLogin,
    LoginStatus
| order by DaysSinceLastLogin asc