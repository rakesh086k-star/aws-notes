let UserUPN = "user@domain.com";

WVDConnectionNetworkData
| join kind=leftouter (
    WVDConnections
    | distinct CorrelationId, UserName
) on CorrelationId
| where UserName =~ UserUPN
| where TimeGenerated >= ago(7d)
| summarize
    Avg_RTT_ms = round(avg(EstRoundTripTimeInMs), 2),
    P95_RTT_ms = round(percentile(EstRoundTripTimeInMs, 95), 2),
    Max_RTT_ms = round(max(EstRoundTripTimeInMs), 2),
    Avg_Bandwidth_KBps = round(avg(EstAvailableBandwidthKBps), 2),
    Min_Bandwidth_KBps = round(min(EstAvailableBandwidthKBps), 2),
    Data_Points = count()
    by bin(TimeGenerated, 30m)
| order by TimeGenerated asc



let UserUPN = "user@domain.com";

WVDConnectionNetworkData
| join kind=leftouter (
    WVDConnections
    | distinct CorrelationId, UserName
) on CorrelationId
| where UserName =~ UserUPN
| where TimeGenerated >= ago(7d)
| summarize
    Avg_RTT_ms = round(avg(EstRoundTripTimeInMs), 2),
    P95_RTT_ms = round(percentile(EstRoundTripTimeInMs, 95), 2),
    Max_RTT_ms = round(max(EstRoundTripTimeInMs), 2),
    Avg_Bandwidth_KBps = round(avg(EstAvailableBandwidthKBps), 2),
    Min_Bandwidth_KBps = round(min(EstAvailableBandwidthKBps), 2),
    Data_Points = count()
    by bin(TimeGenerated, 30m)
| extend
    Status = case(
        P95_RTT_ms <= 100, "Healthy",
        P95_RTT_ms <= 200, "Warning",
        "Critical"
    )
| order by TimeGenerated asc







let UserUPN = "user@domain.com";

let NetworkData =
    WVDConnectionNetworkData
    | join kind=leftouter (
        WVDConnections
        | distinct CorrelationId, UserName, SessionHostName
    ) on CorrelationId
    | where UserName =~ UserUPN
    | where TimeGenerated >= ago(7d)
    | extend TimeSlot = bin(TimeGenerated, 30m)
    | summarize
        Avg_RTT_ms = round(avg(EstRoundTripTimeInMs), 2),
        P95_RTT_ms = round(percentile(EstRoundTripTimeInMs, 95), 2),
        Max_RTT_ms = max(EstRoundTripTimeInMs),
        Avg_Bandwidth_KBps = round(avg(EstAvailableBandwidthKBps), 2),
        Min_Bandwidth_KBps = min(EstAvailableBandwidthKBps)
        by UserName, SessionHostName, TimeSlot;

let HostCPU =
    Perf
    | where TimeGenerated >= ago(7d)
    | where ObjectName == "Processor"
    | where CounterName == "% Processor Time"
    | where InstanceName == "_Total"
    | summarize
        Avg_CPU = round(avg(CounterValue), 2),
        Max_CPU = round(max(CounterValue), 2)
        by Computer, TimeSlot = bin(TimeGenerated, 30m);

let HostMemory =
    Perf
    | where TimeGenerated >= ago(7d)
    | where ObjectName == "Memory"
    | where CounterName == "% Committed Bytes In Use"
    | summarize
        Avg_Memory = round(avg(CounterValue), 2),
        Max_Memory = round(max(CounterValue), 2)
        by Computer, TimeSlot = bin(TimeGenerated, 30m);

NetworkData
| join kind=leftouter HostCPU
    on $left.SessionHostName == $right.Computer,
       $left.TimeSlot == $right.TimeSlot
| join kind=leftouter HostMemory
    on $left.SessionHostName == $right.Computer,
       $left.TimeSlot == $right.TimeSlot
| extend
    Network_Status = case(
        P95_RTT_ms <= 100 and Avg_Bandwidth_KBps >= 1000, "Healthy",
        P95_RTT_ms <= 200 and Avg_Bandwidth_KBps >= 500, "Warning",
        "Critical"
    ),
    Host_Status = case(
        Avg_CPU < 70 and Avg_Memory < 70, "Healthy",
        Avg_CPU < 85 and Avg_Memory < 85, "Warning",
        "Critical"
    )
| extend
    Overall_Status = case(
        Network_Status == "Critical" or Host_Status == "Critical", "Critical",
        Network_Status == "Warning" or Host_Status == "Warning", "Warning",
        "Healthy"
    )
| project
    TimeSlot,
    UserName,
    SessionHostName,
    Avg_RTT_ms,
    P95_RTT_ms,
    Max_RTT_ms,
    Avg_Bandwidth_KBps,
    Min_Bandwidth_KBps,
    Avg_CPU,
    Max_CPU,
    Avg_Memory,
    Max_Memory,
    Network_Status,
    Host_Status,
    Overall_Status
| order by TimeSlot asc






