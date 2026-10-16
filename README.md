
let StartTime = datetime(2026-10-12 00:00:00);
let EndTime = datetime(2026-10-15 23:59:59);
let TargetUser = "Takshu255467@exlservice.com";
let UserHosts =
    WVDConnections
    | where TimeGenerated between (StartTime .. EndTime)
    | where UserName =~ TargetUser
    | where isnotempty(SessionHostName)
    | summarize by SessionHostName;
Perf
| where TimeGenerated between (StartTime .. EndTime)
| where Computer in~ (UserHosts | project SessionHostName)
| where (
    (ObjectName == "Processor"
        and CounterName == "% Processor Time"
        and InstanceName == "_Total")
    or
    (ObjectName == "Memory"
        and CounterName == "% Committed Bytes In Use")
)
| summarize
    CPU_Average_Percent = avgif(
        CounterValue,
        ObjectName == "Processor"
    ),
    CPU_Max_Percent = maxif(
        CounterValue,
        ObjectName == "Processor"
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
| where CPU_Max_Percent >= 90
    or Memory_Max_Percent >= 90
| extend HighUtilizationReason = case(
    CPU_Max_Percent >= 90 and Memory_Max_Percent >= 90,
        "CPU and Memory >= 90%",
    CPU_Max_Percent >= 90,
        "CPU >= 90%",
    "Memory >= 90%"
)
| project
    TimeBucket,
    SessionHost = Computer,
    CPU_Average_Percent,
    CPU_Max_Percent,
    Memory_Average_Percent,
    Memory_Max_Percent,
    HighUtilizationReason
| order by TimeBucket asc
