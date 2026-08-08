LAWCUSTOMfslogix_CL
| where TimeGenerated > ago(24h)
| extend RawText = tostring(RawData)
| extend Severity = extract(@"\[([A-Z]+)\]", 1, RawText)
| extend LogTime = extract(@"^\[([0-9:.]+)\]", 1, RawText)
| extend Message = trim(" ", extract(@"\[INFO\]\s*(.*)", 1, RawText))
| project
    TimeGenerated,
    Computer,
    FilePath,
    Severity,
    LogTime,
    Message
| order by TimeGenerated desc