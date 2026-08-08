Event
| where TimeGenerated > ago(24h)
| where Source has "FSLogix"
| project
    TimeGenerated,
    Computer,
    EventID,
    EventLevelName,
    RenderedDescription
| order by TimeGenerated desc