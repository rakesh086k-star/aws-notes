AzureDiagnostics
| where ResourceProvider == "MICROSOFT.DESKTOPVIRTUALIZATION"
| limit 1



AzureDiagnostics
| where ResourceProvider == "MICROSOFT.DESKTOPVIRTUALIZATION"
| where Message has "error"
    or Message has "fail"
| summarize
    ErrorCount = count(),
    ErrorMessages = make_set(Message, 5)
    by Resource
| extend Priority = case(
    ErrorCount >= 20, "Critical 🔴",
    ErrorCount >= 10, "High 🟠",
    ErrorCount >= 5, "Medium 🟡",
    "Low 🟢"
)
| order by ErrorCount desc