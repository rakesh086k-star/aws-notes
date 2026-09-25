WVDConnectionNetworkData
| where TimeGenerated > ago(5h)
| join kind=inner (
    WVDConnections
    | where State == "Connected"
    | where UserName != ""
) on CorrelationId
| summarize
    AvgRTT = round(avg(EstRoundTripTimeInMs), 2),
    MaxRTT = max(EstRoundTripTimeInMs),
    P90RTT = percentile(EstRoundTripTimeInMs, 90)
    by UserName
| order by AvgRTT desc