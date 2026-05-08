WVDConnections
| summarize ErrorCount = count() by SessionHostName
| extend Priority = case(
    ErrorCount >= 20, "Critical 🔴",
    ErrorCount >= 10, "High 🟠",
    ErrorCount >= 5, "Medium 🟡",
    "Low 🟢"
)
| order by ErrorCount desc