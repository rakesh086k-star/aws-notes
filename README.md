union
(
    WVDConnections
    | where TimeGenerated > ago(1d)
    | summarize TotalLogins = count()
    | extend Period = "Daily"
),
(
    WVDConnections
    | where TimeGenerated > ago(7d)
    | summarize TotalLogins = count()
    | extend Period = "Weekly"
),
(
    WVDConnections
    | where TimeGenerated > ago(30d)
    | summarize TotalLogins = count()
    | extend Period = "Monthly"
)
| order by Period asc