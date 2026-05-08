WVDConnections
| project UserName, CorrelationId, SessionHostName
| join kind=inner (
    WVDErrors
    | project CorrelationId, Code, Message, ServiceError
) on CorrelationId
| summarize
    ErrorCount = count(),
    ErrorCodes = make_set(Code, 3),
    ErrorMessages = make_set(Message, 3)
    by SessionHostName, UserName
| extend Severity = case(
    ErrorCount >= 10, "Critical 🔴",
    ErrorCount >= 5, "High 🟠",
    ErrorCount >= 2, "Medium 🟡",
    "Low 🟢"
)
| order by ErrorCount desc