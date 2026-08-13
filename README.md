LAWCustomFslogix_CL
| where TimeGenerated > ago(24h)
| where strlen(RawData) > 10
| project TimeGenerated, Computer, RawData
| order by TimeGenerated desc
| take 100


Event
| where TimeGenerated > ago(24h)
| where EventLog has "FSLogix"
   or Source has "FSLogix"
   or RenderedDescription has "FSLogix"
| project
    TimeGenerated,
    Computer,
    EventLog,
    Source,
    EventID,
    EventLevelName,
    RenderedDescription
| order by TimeGenerated desc
| take 100



Event
| where TimeGenerated > ago(48h)
| where EventLog has "FSLogix"
    or Source has "FSLogix"
    or RenderedDescription has "FSLogix"
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
| where TimeGenerated > ago(48h)
| where EventLog has "FSLogix"
    or Source has "FSLogix"
    or RenderedDescription has "FSLogix"
| where EventLevelName in ("Error", "Warning")
| project
    TimeGenerated,
    Computer,
    EventID,
    EventLevelName,
    EventLog,
    Source,
    RenderedDescription,
    UserName
| order by TimeGenerated desc




Event
| where TimeGenerated > ago(48h)
| where EventLog has "FSLogix"
    or Source has "FSLogix"
    or RenderedDescription has "FSLogix"
| where RenderedDescription has_any ("profile", "size", "MB", "GB", "VHD", "VHDX")
| project
    TimeGenerated,
    Computer,
    UserName,
    EventID,
    EventLevelName,
    RenderedDescription
| order by TimeGenerated desc


Event
| where TimeGenerated > ago(48h)
| where EventLog has "FSLogix"
    or Source has "FSLogix"
    or RenderedDescription has "FSLogix"
| where RenderedDescription has_any ("profile", "size", "MB", "GB", "VHD", "VHDX")
| project
    TimeGenerated,
    Computer,
    UserName,
    EventID,
    EventLevelName,
    RenderedDescription
| order by TimeGenerated desc



Event
| where TimeGenerated > ago(48h)
| where EventLog has "FSLogix"
    or Source has "FSLogix"
    or RenderedDescription has "FSLogix"
| extend Description = tostring(RenderedDescription)
| project
    TimeGenerated,
    Computer,
    UserName,
    EventID,
    EventLevelName,
    EventLog,
    Source,
    Description
| order by TimeGenerated desc
| take 500




AALW_Custom_FSLogix_CLR
| where TimeGenerated > ago(48h)
| project TimeGenerated, ComputerName, FilePath, RowData
| where RowData has_any ("size", "Size", "profile", "Profile", "VHDX", "VHD")
| order by TimeGenerated desc
| take 100




