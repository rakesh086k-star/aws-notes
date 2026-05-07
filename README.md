AzureDiagnostics
| where ResourceProvider == "MICROSOFT.DESKTOPVIRTUALIZATION"
| where Message contains "error"
    or Message contains "fail"
| summarize
    ErrorCount = count(),
    ErrorMessages = make_set(Message, 5)
    by Computer
| extend Priority = case(
    ErrorCount >= 20, "Critical 🔴",
    ErrorCount >= 10, "High 🟠",
    ErrorCount >= 5, "Medium 🟡",
    "Low 🟢"
)
| order by ErrorCount desc