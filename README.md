Event
| where TimeGenerated > ago(24h)
| where Source has "FSLogix"
| where RenderedDescription has_any (
    "SizeInMBs",
    "MB left",
    "% free",
    "Free space",
    "Profile size"
)
| project
    TimeGenerated,
    Computer,
    EventID,
    EventLevelName,
    RenderedDescription
| order by TimeGenerated desc 