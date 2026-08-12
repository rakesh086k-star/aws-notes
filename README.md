let SelectedUser = "";   // User name डालें, blank = सभी users

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
| summarize arg_max(TimeGenerated, UserName, SessionHostName)
    by CorrelationId
| where isempty(SelectedUser) or UserName =~ SelectedUser
| extend HostKey = tolower(tostring(split(SessionHostName, ".")[0]));

let HostCPU =
Perf
| where TimeGenerated >= ago(7d)
| where ObjectName == "Processor"
| where CounterName == "% Processor Time"
| where InstanceName == "_Total"
| extend
    TimeSlot = bin(TimeGenerated, 30m),
    HostKey = tolower(tostring(split(Computer, ".")[0]))
| summarize
    Avg_CPU_Percent = round(avg(CounterValue), 2),
    Max_CPU_Percent = round(max(CounterValue), 2)
    by HostKey, TimeSlot;

let HostMemory =
Perf
| where TimeGenerated >= ago(7d)
| where ObjectName == "Memory"
| where CounterName == "% Committed Bytes In Use"
| extend
    TimeSlot = bin(TimeGenerated, 30m),
    HostKey = tolower(tostring(split(Computer, ".")[0]))
| summarize
    Avg_Memory_Percent = round(avg(CounterValue), 2),
    Max_Memory_Percent = round(max(CounterValue), 2)
    by HostKey, TimeSlot;

NetworkData
| join kind=leftouter Connections on CorrelationId
| join kind=leftouter HostCPU
    on $left.HostKey == $right.HostKey
    and $left.TimeSlot == $right.TimeSlot
| join kind=leftouter HostMemory
    on $left.HostKey == $right.HostKey
    and $left.TimeSlot == $right.TimeSlot
| extend
    RTT_Avg =
        iff(isnull(Avg_RTT_ms), "N/A",
            strcat(tostring(Avg_RTT_ms), " ms")),

    RTT_Max =
        iff(isnull(Max_RTT_ms), "N/A",
            strcat(tostring(Max_RTT_ms), " ms")),

    Bandwidth_Avg =
        iff(isnull(Avg_Bandwidth_KBps), "N/A",
            iff(Avg_Bandwidth_KBps >= 1024,
                strcat(tostring(round(Avg_Bandwidth_KBps * 8 / 1024, 2)), " Mbps"),
                strcat(tostring(round(Avg_Bandwidth_KBps, 2)), " KBps"))),

    Bandwidth_Min =
        iff(isnull(Min_Bandwidth_KBps), "N/A",
            iff(Min_Bandwidth_KBps >= 1024,
                strcat(tostring(round(Min_Bandwidth_KBps * 8 / 1024, 2)), " Mbps"),
                strcat(tostring(round(Min_Bandwidth_KBps, 2)), " KBps"))),

    CPU_Avg =
        iff(isnull(Avg_CPU_Percent), "N/A",
            strcat(tostring(Avg_CPU_Percent), " %")),

    CPU_Max =
        iff(isnull(Max_CPU_Percent), "N/A",
            strcat(tostring(Max_CPU_Percent), " %")),

    Memory_Avg =
        iff(isnull(Avg_Memory_Percent), "N/A",
            strcat(tostring(Avg_Memory_Percent), " %")),

    Memory_Max =
        iff(isnull(Max_Memory_Percent), "N/A",
            strcat(tostring(Max_Memory_Percent), " %")),

    Network_Health =
        case(
            Avg_RTT_ms > 200 or Avg_Bandwidth_KBps < 500, "Critical",
            Avg_RTT_ms > 100 or Avg_Bandwidth_KBps < 1024, "Warning",
            "Healthy"
        )
| project
    TimeSlot,
    UserName,
    RTT_Avg,
    RTT_Max,
    Bandwidth_Avg,
    Bandwidth_Min,
    CPU_Avg,
    CPU_Max,
    Memory_Avg,
    Memory_Max,
    Network_Health
| order by TimeSlot asc