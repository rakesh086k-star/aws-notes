Perf
| where TimeGenerated >= ago(7d)
| where ObjectName == "Processor"
| where CounterName == "% Processor Time"
| where InstanceName == "_Total"
| summarize
    CPU_Avg = round(avg(CounterValue), 2),
    CPU_Max = round(max(CounterValue), 2)
    by Computer, bin(TimeGenerated, 30m)
| order by TimeGenerated asc