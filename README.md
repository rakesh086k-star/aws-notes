WVDConnections
| where TimeGenerated > ago(30d)
| extend TimeBucket = case(
    TimeGenerated > ago(1d), "Daily (24h)",
    TimeGenerated > ago(7d), "Weekly (7d)",
    TimeGenerated > ago(30d), "Monthly (30d)",
    "Older"
)
| summarize UniqueUsers = dcount(UserName) by TimeBucket
| order by TimeBucket desc