WVDConnections
| extend TimeBucket = case(
    TimeGenerated > ago(1d), "Daily",
    TimeGenerated > ago(7d), "Weekly",
    TimeGenerated > ago(30d), "Monthly",
    "Older"
)
| summarize TotalLogins = count() by TimeBucket
| order by TimeBucket desc