Perf
| where ObjectName == "Memory"
| where CounterName == "% Committed Bytes In Use"
| where TimeGenerated {TimeRange}
| summarize AvgMemory = avg(CounterValue) by Computer
| where AvgMemory > 80
| project Computer, AvgMemory