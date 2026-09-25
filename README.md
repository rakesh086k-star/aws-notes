let NetworkData =
    WVDConnectionNetworkData
    | where TimeGenerated > ago(5h)
    | summarize
        AvgRTT = avg(EstRoundTripTimeInMs),
        MaxRTT = max(EstRoundTripTimeInMs),
        P90RTT = percentile(EstRoundTripTimeInMs, 90)
        by CorrelationId;

NetworkData
| join kind=inner (
    WVDConnections
    | where State == "Connected"
    | where UserName != ""
    | project CorrelationId, UserName
) on CorrelationId
| summarize
    AvgRTT = round(avg(AvgRTT), 2),
    MaxRTT = max(MaxRTT),
    P90RTT = percentile(P90RTT, 90)
    by UserName
| order by AvgRTT desc