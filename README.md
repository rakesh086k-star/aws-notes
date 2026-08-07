Event
| where TimeGenerated > ago(24h)
| where Source has "FSLogix"
| project
    TimeGenerated,
    Computer,
    EventLog,
    Source,
    EventID,
    EventLevelName,
    RenderedDescription
| order by TimeGenerated desc