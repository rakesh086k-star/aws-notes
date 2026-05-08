WVDConnections
| where TimeGenerated > ago(24h)
| where State in ("Disconnected", "Failed")
| summarize
    TotalIssues = count(),
    States = make_set(State, 5),
    LastIssueTime = max(TimeGenerated)
    by SessionHostName
| extend Severity = case(
    TotalIssues >= 15, "Critical 🔴",
    TotalIssues >= 8, "High 🟠",
    TotalIssues >= 3, "Medium 🟡",
    "Low 🟢"
)
| order by TotalIssues desc