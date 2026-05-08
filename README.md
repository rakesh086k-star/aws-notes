WVDConnections
| where TimeGenerated > ago(24h)
| summarize TotalEvents=count() by SessionHostName
| extend Severity = case(
    TotalEvents >= 15, "Critical 🔴",
    TotalEvents >= 8, "High 🟠",
    TotalEvents >= 3, "Medium 🟡",
    "Low 🟢"
)
| order by TotalEvents desc