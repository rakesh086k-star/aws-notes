WVDConnections
| where TimeGenerated > ago(24h)
| where State == "Completed"
| where ConnectionType == "RDP"
| where isnotempty(FailureType)
    or isnotempty(FailureMessage)
| summarize
    ErrorCount = count(),
    LatestError = max(TimeGenerated),
    FailureReasons = make_set(FailureType, 5),
    FailureMessages = make_set(FailureMessage, 5)
    by SessionHostName
| extend Severity = case(
    ErrorCount >= 15, "Critical 🔴",
    ErrorCount >= 8, "High 🟠",
    ErrorCount >= 3, "Medium 🟡",
    "Low 🟢"
)
| order by ErrorCount desc