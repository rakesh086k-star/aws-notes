WVDConnections
| where TimeGenerated > ago(24h)
| where State != "Connected"
| summarize
    ErrorCount = count(),
    LatestIssue = max(TimeGenerated)
    by SessionHostName, State
| extend Severity = case(
    ErrorCount >= 15, "Critical 🔴",
    ErrorCount >= 8, "High 🟠",
    ErrorCount >= 3, "Medium 🟡",
    "Low 🟢"
)
| order by ErrorCount desc