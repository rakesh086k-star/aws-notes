let SelectedUser = "";   // Blank = all users
                          // Example: "Indresh340697@exlservice.com"

let NetworkData =
WVDConnectionNetworkData
| where TimeGenerated >= ago(7d)
| extend TimeSlot = bin(TimeGenerated, 30m)
| summarize
    Avg_RTT_ms = round(avg(EstRoundTripTimeInMs), 2),
    Max_RTT_ms = round(max(EstRoundTripTimeInMs), 2),
    Avg_Bandwidth_KBps = round(avg(EstAvailableBandwidthKBps), 2),
    Max_Bandwidth_KBps = round(max(EstAvailableBandwidthKBps), 2)
    by CorrelationId, TimeSlot;

let UserMapping =
WVDConnections
| where TimeGenerated >= ago(7d)
| where isnotempty(UserName)
| distinct CorrelationId, UserName
| where isempty(SelectedUser) or UserName =~ SelectedUser;

NetworkData
| join kind=inner UserMapping on CorrelationId
| extend
    RTT_Avg =
        strcat(tostring(Avg_RTT_ms), " ms"),

    RTT_Max =
        strcat(tostring(Max_RTT_ms), " ms"),

    Bandwidth_Avg =
        iff(
            Avg_Bandwidth_KBps >= 1024,
            strcat(tostring(round(Avg_Bandwidth_KBps / 1024.0, 2)), " MBps"),
            strcat(tostring(round(Avg_Bandwidth_KBps, 2)), " KBps")
        ),

    Bandwidth_Max =
        iff(
            Max_Bandwidth_KBps >= 1024,
            strcat(tostring(round(Max_Bandwidth_KBps / 1024.0, 2)), " MBps"),
            strcat(tostring(round(Max_Bandwidth_KBps, 2)), " KBps")
        )
| project
    TimeSlot,
    UserName,
    RTT_Avg,
    RTT_Max,
    Bandwidth_Avg,
    Bandwidth_Max
| order by TimeSlot asc