Perf
| where TimeGenerated >= ago(7d)
| where ObjectName == "Processor Information"
| where CounterName == "% Processor Time"
| where InstanceName == "_Total"
| extend TimeSlot = bin(TimeGenerated, 30m)
| summarize
    CPU_Avg = round(avg(todouble(CounterValue)), 2),
    CPU_Max = round(max(todouble(CounterValue)), 2)
    by Computer, TimeSlot
| project
    TimeSlot,
    Computer,
    CPU_Avg = strcat(tostring(CPU_Avg), " %"),
    CPU_Max = strcat(tostring(CPU_Max), " %")
| order by TimeSlot asc