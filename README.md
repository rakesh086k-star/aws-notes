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
| extend SortOrder = case(
    Period == "Daily", 1,
    Period == "Weekly", 2,
    Period == "Monthly", 3,
    99
)
| project Period, UniqueUsers, SortOrder
| order by SortOrder asc