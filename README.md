Perf
| where TimeGenerated >= ago(1h)
| where ObjectName contains "Processor"
| summarize Records = count() by ObjectName, CounterName, InstanceName
| order by Records desc