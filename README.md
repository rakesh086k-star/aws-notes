Event
| where TimeGenerated > ago(7d)
| where Source has "FSLogix"
| where EventID == 35
| summarize arg_max(TimeGenerated, *) by Computer
| project
    TimeGenerated,
    Computer,
    EventID,
    EventLevelName,
    RenderedDescription
| order by Computer asc


Event
| where TimeGenerated > ago(7d)
| where Source has "FSLogix"
| where EventLevelName == "Error"
| summarize ErrorCount=count(), LastSeen=max(TimeGenerated) by Computer, EventID, RenderedDescription
| order by ErrorCount desc




Event
| where TimeGenerated > ago(7d)
| where Source has "FSLogix"
| where EventLevelName == "Warning"
| summarize WarningCount=count(), LastSeen=max(TimeGenerated) by Computer, EventID, RenderedDescription
| order by WarningCount desc


Event
| where TimeGenerated > ago(7d)
| where Source has "FSLogix"
| where RenderedDescription has_any (
    "Username:",
    "Username",
    "Profile load",
    "Orphaned OST",
    "VHDX"
)
| project
    TimeGenerated,
    Computer,
    EventID,
    EventLevelName,
    RenderedDescription
| order by TimeGenerated desc



Event
| where TimeGenerated > ago(7d)
| where Source has "FSLogix"
| where RenderedDescription has "Profile load"
| project
    TimeGenerated,
    Computer,
    EventID,
    EventLevelName,
    RenderedDescription
| order by TimeGenerated desc





Event
| where TimeGenerated > ago(7d)
| where Source has "FSLogix"
| where RenderedDescription has_any (
    "less than 2 GB",
    "remaining",
    "VHDX"
)
| project
    TimeGenerated,
    Computer,
    EventID,
    EventLevelName,
    RenderedDescription
| order by TimeGenerated desc


Event
| where TimeGenerated > ago(7d)
| where Source has "FSLogix"
| where RenderedDescription has "Orphaned OST"
| summarize
    Count=count(),
    LastSeen=max(TimeGenerated)
    by Computer, EventID, RenderedDescription
| order by Count desc





Event
| where TimeGenerated > ago(7d)
| where Source has "FSLogix"
| summarize Count=count() by EventLevelNam




Event
| where TimeGenerated > ago(7d)
| where Source has "FSLogix"
| where EventLevelName in ("Error", "Warning")
| summarize
    TotalIssues=count(),
    Errors=countif(EventLevelName == "Error"),
    Warnings=countif(EventLevelName == "Warning"),
    LastSeen=max(TimeGenerated)
    by Computer
| order by TotalIssues desc









