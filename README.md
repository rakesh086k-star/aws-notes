Event
| where TimeGenerated > ago(24h)
| where Source has "FSLogix"
| summarize Count = count() by Source
| order by Count desc