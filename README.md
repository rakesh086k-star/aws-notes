WVDConnectionNetworkData
| where TimeGenerated >= ago(5h)
| join kind=inner (
    WVDConnections
    | where State == "Connected"
    | where UserName != ""
    | distinct CorrelationId, UserName, SessionHostName
) on CorrelationId
| extend ComputerName = tostring(SessionHostName)
| summarize
    ["Avg. RTT"] = round(avg(EstRoundTripTimeInMs), 0),
    ["Max. RTT"] = max(EstRoundTripTimeInMs),
    ["P90 RTT"] = percentile(EstRoundTripTimeInMs, 90),
    ["Avg. Bandwidth"] = round(avg(EstAvailableBandwidthKBps), 0),
    ["Max. Bandwidth"] = max(EstAvailableBandwidthKBps),
    ["P90 Bandwidth"] = percentile(EstAvailableBandwidthKBps, 90)
    by UserName, ComputerName
| order by ["Avg. RTT"] desc