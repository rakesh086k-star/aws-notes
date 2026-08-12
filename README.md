AzureMetrics
| where TimeGenerated >= ago(7d)
| summarize Count=count() by MetricName
| order by Count desc



let SelectedUser = "";   
// Blank = all users
// Example:
// let SelectedUser = "Anuradha218532@exlservice.com";

WVDConnectionNetworkData
| where TimeGenerated >= ago(7d)
| extend TimeSlot = bin(TimeGenerated, 30m)
| join kind=inner (
    WVDConnections
    | where TimeGenerated >= ago(7d)
    | where State == "Connected"
    | where UserName != ""
    | distinct CorrelationId, UserName
) on CorrelationId
| where isempty(SelectedUser) or UserName =~ SelectedUser
| summarize
    Avg_RTT = round(avg(EstRoundTripTimeInMs), 2),
    Max_RTT = round(max(EstRoundTripTimeInMs), 2),
    Avg_Bandwidth_KBps = round(avg(EstAvailableBandwidthKBps), 2),
    Max_Bandwidth_KBps = round(max(EstAvailableBandwidthKBps), 2)
    by UserName, TimeSlot
| extend
    Network_Health =
        case(
            Avg_RTT > 200 or Avg_Bandwidth_KBps < 500,
            "Critical",

            Avg_RTT > 100 or Avg_Bandwidth_KBps < 1024,
            "Warning",

            "Healthy"
        )
| extend
    Avg_RTT_Display = strcat(tostring(Avg_RTT), " ms"),
    Max_RTT_Display = strcat(tostring(Max_RTT), " ms"),

    Avg_Bandwidth_Display =
        iff(
            Avg_Bandwidth_KBps >= 1024,
            strcat(
                tostring(round(Avg_Bandwidth_KBps / 1024.0, 2)),
                " MBps"
            ),
            strcat(
                tostring(round(Avg_Bandwidth_KBps, 2)),
                " KBps"
            )
        ),

    Max_Bandwidth_Display =
        iff(
            Max_Bandwidth_KBps >= 1024,
            strcat(
                tostring(round(Max_Bandwidth_KBps / 1024.0, 2)),
                " MBps"
            ),
            strcat(
                tostring(round(Max_Bandwidth_KBps, 2)),
                " KBps"
            )
        )
| project
    TimeSlot,
    UserName,
    Avg_RTT = Avg_RTT_Display,
    Max_RTT = Max_RTT_Display,
    Avg_Bandwidth = Avg_Bandwidth_Display,
    Max_Bandwidth = Max_Bandwidth_Display,
    Network_Health
| order by TimeSlot asc