
let StartTime = datetime(2026-10-12 00:00:00);
let EndTime = datetime(2026-10-15 23:59:59);
let TargetUser = "Takshu255467@exlservice.com";
let UserHosts =
    WVDConnections
    | where TimeGenerated between (StartTime .. EndTime)
    | where UserName =~ TargetUser
    | where isnotempty(SessionHostName)
    | distinct SessionHostName;
Perf
| where TimeGenerated between (StartTime .. EndTime)
| where Computer in~ (UserHosts)
| where (
    (CounterName == "% Processor Time"
        and InstanceName in~ ("_Total", "Total"))
    or
    (ObjectName == "Memory"
        and CounterName == "% Committed Bytes In Use")
)
| summarize
    CPU_Average_Percent = avgif(
        CounterValue,
        CounterName == "% Processor Time"
    ),
    CPU_Max_Percent = maxif(
        CounterValue,
        CounterName == "% Processor Time"
    ),
    Memory_Average_Percent = avgif(
        CounterValue,
        ObjectName == "Memory"
    ),
    Memory_Max_Percent = maxif(
        CounterValue,
        ObjectName == "Memory"
    )
    by Computer, TimeBucket = bin(TimeGenerated, 30m)
| extend
    CPU_Average_Percent = round(CPU_Average_Percent, 2),
    CPU_Max_Percent = round(CPU_Max_Percent, 2),
    Memory_Average_Percent = round(Memory_Average_Percent, 2),
    Memory_Max_Percent = round(Memory_Max_Percent, 2)
| where CPU_Max_Percent > 90
    or Memory_Max_Percent > 90
| project
    TimeBucket,
    SessionHost = Computer,
    CPU_Average_Percent,
    CPU_Max_Percent,
    Memory_Average_Percent,
    Memory_Max_Percent
| order by TimeBucket asc
