WVDConnectionNetworkData
| where TimeGenerated > ago(5h)
| join kind=inner (
    WVDConnections
    | where State == "Connected"
    | where UserName != ""
    | project CorrelationId, UserName
) on CorrelationId
| project UserName, EstRoundTripTimeInMs
| summarize
    AvgRTT = avg(EstRoundTripTimeInMs),
    MaxRTT = max(EstRoundTripTimeInMs),
    P90RTT = percentile(EstRoundTripTimeInMs, 90)
    by UserName
| order by AvgRTT desc