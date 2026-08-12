Perf
| where TimeGenerated >= ago(7d)
| summarize Count=count() by ObjectName, CounterName
| order by Count desc