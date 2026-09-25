WVDConnectionNetworkData
| where TimeGenerated > ago(5h)
| join kind=inner (
    WVDConnections
    | where State == "Connected"
    | where UserName != ""
) on CorrelationId
| summarize
    AvgRTT = avg(EstRoundTripTimeInMs),
    MaxRTT = max(EstRoundTripTimeInMs)
    by UserName