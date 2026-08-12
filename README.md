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