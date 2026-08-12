let SelectedUser = "";   // Blank = all users
                          // Example: "Indresh340697@exlservice.com"

let NetworkData =
WVDConnectionNetworkData
| where TimeGenerated >= ago(7d)
| extend TimeSlot = bin(TimeGenerated, 30m)
| summarize
    Avg_RTT = round(avg(EstRoundTripTimeInMs), 2),
    Max_RTT = round(max(EstRoundTripTimeInMs), 2),
    Avg_BW_KBps = round(avg(EstAvailableBandwidthKBps), 2),
    Min_BW_KBps = round(min(EstAvailableBandwidthKBps), 2)
    by CorrelationId, TimeSlot;

let UserMapping =
WVDConnections
| where TimeGenerated >= ago(7d)
| where isnotempty(UserName)
| distinct CorrelationId, UserName, SessionHostName
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
    CPU_Avg = round(avg(CounterValue), 2),
    CPU_Max = round(max(CounterValue), 2)
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
    Memory_Avg = round(avg(CounterValue), 2),
    Memory_Max = round(max(CounterValue), 2)
    by HostKey, TimeSlot;

NetworkData
| join kind=inner UserMapping on CorrelationId
| join kind=leftouter HostCPU
    on HostKey, TimeSlot
| join kind=leftouter HostMemory
    on HostKey, TimeSlot
| extend
    RTT_Avg_Display =
        strcat(tostring(Avg_RTT), " ms"),

    RTT_Max_Display =
        strcat(tostring(Max_RTT), " ms"),

    Bandwidth_Avg_Display =
        iff(
            Avg_BW_KBps >= 1024,
            strcat(tostring(round(Avg_BW_KBps / 1024.0, 2)), " MBps"),
            strcat(tostring(round(Avg_BW_KBps, 2)), " KBps")
        ),

    Bandwidth_Min_Display =
        iff(
            Min_BW_KBps >= 1024,
            strcat(tostring(round(Min_BW_KBps / 1024.0, 2)), " MBps"),
            strcat(tostring(round(Min_BW_KBps, 2)), " KBps")
        ),

    CPU_Avg_Display =
        iff(isnull(CPU_Avg), "N/A",
            strcat(tostring(CPU_Avg), " %")),

    CPU_Max_Display =
        iff(isnull(CPU_Max), "N/A",
            strcat(tostring(CPU_Max), " %")),

    Memory_Avg_Display =
        iff(isnull(Memory_Avg), "N/A",
            strcat(tostring(Memory_Avg), " %")),

    Memory_Max_Display =
        iff(isnull(Memory_Max), "N/A",
            strcat(tostring(Memory_Max), " %")),

    Network_Health =
        case(
            Avg_RTT > 200 or Avg_BW_KBps < 500, "Critical",
            Avg_RTT > 100 or Avg_BW_KBps < 1024, "Warning",
            "Healthy"
        )
| project
    TimeSlot,
    UserName,
    RTT_Avg = RTT_Avg_Display,
    RTT_Max = RTT_Max_Display,
    Bandwidth_Avg = Bandwidth_Avg_Display,
    Bandwidth_Min = Bandwidth_Min_Display,
    CPU_Avg = CPU_Avg_Display,
    CPU_Max = CPU_Max_Display,
    Memory_Avg = Memory_Avg_Display,
    Memory_Max = Memory_Max_Display,
    Network_Health
| order by TimeSlot asc