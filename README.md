union
(
    WVDConnections
    | where TimeGenerated > ago(1d)
    | summarize UniqueUsers = dcount(UserName)
    | extend Period = "Daily"
),
(
    WVDConnections
    | where TimeGenerated > ago(7d)
    | summarize UniqueUsers = dcount(UserName)
    | extend Period = "Weekly"
),
(
    WVDConnections
    | where TimeGenerated > ago(30d)
    | summarize UniqueUsers = dcount(UserName)
    | extend Period = "Monthly"
)
| order by Period asc