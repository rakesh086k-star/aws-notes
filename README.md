
let StartTime = datetime(2026-10-12 00:00:00);
let EndTime = datetime(2026-10-16 23:59:59);
let TargetHost = "REPLACE-WITH-SESSION-HOST-NAME";
Perf
| where TimeGenerated between (StartTime .. EndTime)
| where Computer =~ TargetHost
| summarize
    CPU_Avg_Percent = round(avgif(
        CounterValue,
        ObjectName == "Processor"
        and CounterName == "% Processor Time"
        and InstanceName == "_Total"
    ), 2),
    CPU_Max_Percent = round(maxif(
        CounterValue,
        ObjectName == "Processor"
        and CounterName == "% Processor Time"
        and InstanceName == "_Total"
    ), 2),
    AvailableMemory_Avg_MB = round(avgif(
        CounterValue,
        ObjectName == "Memory"
        and CounterName == "Available MBytes"
    ), 2),
    AvailableMemory_Min_MB = round(minif(
        CounterValue,
        ObjectName == "Memory"
        and CounterName == "Available MBytes"
    ), 2)
    by Computer, bin(TimeGenerated, 5m)
| order by TimeGenerated asc
