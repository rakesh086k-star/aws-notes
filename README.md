let CPU =
Perf
| where TimeGenerated >= ago(7d)
| where ObjectName == "Processor Information"
| where CounterName == "% Processor Time"
| where InstanceName == "_Total"
| extend TimeSlot = bin(TimeGenerated, 30m)
| summarize
    CPU_Avg = round(avg(todouble(CounterValue)), 2),
    CPU_Max = round(max(todouble(CounterValue)), 2)
    by Computer, TimeSlot;

let Memory =
Perf
| where TimeGenerated >= ago(7d)
| where ObjectName == "Memory"
| where CounterName == "Available MBytes"
| extend TimeSlot = bin(TimeGenerated, 30m)
| summarize
    Memory_Avg_MB = round(avg(todouble(CounterValue)), 2),
    Memory_Min_MB = round(min(todouble(CounterValue)), 2)
    by Computer, TimeSlot;

CPU
| join kind=leftouter Memory
    on Computer, TimeSlot
| extend
    CPU_Avg_Display = strcat(tostring(CPU_Avg), " %"),
    CPU_Max_Display = strcat(tostring(CPU_Max), " %"),
    Memory_Avg_Display = strcat(tostring(Memory_Avg_MB), " MB"),
    Memory_Min_Display = strcat(tostring(Memory_Min_MB), " MB")
| extend
    Health_Status =
        case(
            CPU_Max >= 90 or Memory_Min_MB < 2048, "Critical",
            CPU_Max >= 75 or Memory_Min_MB < 4096, "Warning",
            "Healthy"
        )
| project
    TimeSlot,
    Computer,
    CPU_Avg = CPU_Avg_Display,
    CPU_Max = CPU_Max_Display,
    Available_Memory_Avg = Memory_Avg_Display,
    Available_Memory_Min = Memory_Min_Display,
    Health_Status
| order by TimeSlot asc