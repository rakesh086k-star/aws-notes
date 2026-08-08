LAWCUSTOMfslogix_CL
| where TimeGenerated > ago(24h)
| mv-expand RawData
| extend RawText = tostring(RawData)
| extend LogTime = extract(@"^\[([0-9:.]+)\]", 1, RawText)
| extend Severity = extract(@"\[(INFO|ERROR|WARN|WARNING|DEBUG)\]", 1, RawText)
| extend Message = extract(@"\]\s*\[(INFO|ERROR|WARN|WARNING|DEBUG)\]\s*(.*)$", 2, RawText)
| project
    TimeGenerated,
    Computer,
    FilePath,
    LogTime,
    Severity,
    Message
| order by TimeGenerated desc