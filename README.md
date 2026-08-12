let NetworkData =
WVDConnectionNetworkData
| where TimeGenerated >= ago(7d)
| extend TimeSlot = bin(TimeGenerated, 30m)
| summarize
    Avg_RTT_ms = round(avg(EstRoundTripTimeInMs), 2),
    Max_RTT_ms = round(max(EstRoundTripTimeInMs), 2),
    Avg_Bandwidth_KBps = round(avg(EstAvailableBandwidthKBps), 2),
    Min_Bandwidth_KBps = round(min(EstAvailableBandwidthKBps), 2)
    by CorrelationId, TimeSlot;

let Connections =
WVDConnections
| where TimeGenerated >= ago(7d)
| summarize arg_max(TimeGenerated, UserName, SessionHostName) by CorrelationId;

let HostCPU =
Perf
| where TimeGenerated >= ago(7d)
| where ObjectName == "Processor"
| where CounterName == "% Processor Time"
| where InstanceName == "_Total"
| extend TimeSlot = bin(TimeGenerated, 30m)
| summarize
    Avg_CPU_Percent = round(avg(CounterValue), 2),
    Max_CPU_Percent = round(max(CounterValue), 2)
    by Computer, TimeSlot;

let HostMemory =
Perf
| where TimeGenerated >= ago(7d)
| where ObjectName == "Memory"
| where CounterName == "% Committed Bytes In Use"
| extend TimeSlot = bin(TimeGenerated, 30m)
| summarize
    Avg_Memory_Percent = round(avg(CounterValue), 2),
    Max_Memory_Percent = round(max(CounterValue), 2)
    by Computer, TimeSlot;

NetworkData
| join kind=leftouter Connections on CorrelationId
| join kind=leftouter HostCPU
    on $left.SessionHostName == $right.Computer
    and $left.TimeSlot == $right.TimeSlot
| join kind=leftouter HostMemory
    on $left.SessionHostName == $right.Computer
    and $left.TimeSlot == $right.TimeSlot
| project
    TimeSlot,
    UserName,
    RTT_Avg = strcat(tostring(Avg_RTT_ms), " ms"),
    RTT_Max = strcat(tostring(Max_RTT_ms), " ms"),
    Bandwidth_Avg = strcat(tostring(Avg_Bandwidth_KBps), " KBps"),
    Bandwidth_Min = strcat(tostring(Min_Bandwidth_KBps), " KBps"),
    CPU_Avg = strcat(tostring(Avg_CPU_Percent), " %"),
    CPU_Max = strcat(tostring(Max_CPU_Percent), " %"),
    Memory_Avg = strcat(tostring(Avg_Memory_Percent), " %"),
    Memory_Max = strcat(tostring(Max_Memory_Percent), " %")
| order by TimeSlot asc