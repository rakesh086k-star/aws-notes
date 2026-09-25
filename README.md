WVDConnectionNetworkData
| where TimeGenerated > ago(5h)
| summarize
    AvgRTT = avg(EstRoundTripTimeInMs),
    P90RTT = percentile(EstRoundTripTimeInMs, 90)