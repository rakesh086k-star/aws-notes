Perf
| where TimeGenerated >= ago(7d)
| where ObjectName == "Processor"
| where CounterName == "% Processor Time"
| where InstanceName == "_Total"
| extend TimeSlot = bin(TimeGenerated, 30m)
| summarize
    CPU_Avg = round(avg(CounterValue), 2),
    CPU_Max = round(max(CounterValue), 2)
    by Computer, TimeSlot
| order by TimeSlot asc