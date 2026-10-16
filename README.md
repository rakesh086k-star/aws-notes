
let StartTime = datetime(2026-10-12 00:00:00);
let EndTime = datetime(2026-10-15 23:59:59);
let TargetUser = "Takshu255467@exlservice.com";

// Get Takshu's connection errors
let UserErrors =
    WVDErrors
    | where TimeGenerated between (StartTime .. EndTime)
    | where UserName =~ TargetUser
    | project
        ErrorTime = TimeGenerated,
        UserName,
        ErrorCode,
        ErrorMessage = Message,
        SessionHostName;

// Get session host CPU and memory metrics
let HostPerformance =
    Perf
    | where TimeGenerated between (StartTime .. EndTime)
    | where (
        (ObjectName == "Processor"
            and CounterName == "% Processor Time"
            and InstanceName == "_Total")
        or
        (ObjectName == "Memory"
            and CounterName == "Available MBytes")
    )
    | summarize
        CPU_Avg_Percent = round(avgif(
            CounterValue,
            ObjectName == "Processor"
        ), 2),
        CPU_Max_Percent = round(maxif(
            CounterValue,
            ObjectName == "Processor"
        ), 2),
        AvailableMemory_Avg_MB = round(avgif(
            CounterValue,
            ObjectName == "Memory"
        ), 2),
        AvailableMemory_Min_MB = round(minif(
            CounterValue,
            ObjectName == "Memory"
        ), 2)
        by Computer, TimeBucket = bin(TimeGenerated, 5m);

// Match errors to performance data around the same time
UserErrors
| extend TimeBucket = bin(ErrorTime, 5m)
| join kind=leftouter HostPerformance
    on $left.SessionHostName == $right.Computer,
       $left.TimeBucket == $right.TimeBucket
| extend AvailableMemory_Min_GB =
    round(AvailableMemory_Min_MB / 1024.0, 2)
| project
    ErrorTime,
    UserName,
    SessionHostName,
    ErrorCode,
    ErrorMessage,
    CPU_Avg_Percent,
    CPU_Max_Percent,
    AvailableMemory_Avg_MB,
    AvailableMemory_Min_MB,
    AvailableMemory_Min_GB
| order by ErrorTime asc
