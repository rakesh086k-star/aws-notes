Perf
| where ObjectName == "Processor"
| where CounterName == "% Processor Time"
| where InstanceName == "_Total"
| summarize CPU_Usage = avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
| sort by TimeGenerated desc


Perf
| where ObjectName == "Memory"
| where CounterName == "% Committed Bytes In Use"
| summarize Memory_Usage = avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
| sort by TimeGenerated desc

