Event
| where TimeGenerated > ago(24h)
| where Source has "FSLogix"
| where EventLevelName in ("Error", "Warning")
| extend
    UserName = extract(@"User:\s*([^\s,]+)", 1, RenderedDescription),
    SessionID = extract(@"SessionId[=: ]+([0-9]+)", 1, RenderedDescription)
| project
    TimeGenerated,
    UserName,
    SessionID,
    SessionHost = Computer,
    ProfileSizeGB = "",
    MaximumSizeGB = "",
    UsagePercentage = "",
    EventID,
    Severity = EventLevelName,
    Error = iff(EventLevelName == "Error", "FSLogix Error", "FSLogix Warning"),
    ErrorDetails = RenderedDescription
| order by TimeGenerated desc