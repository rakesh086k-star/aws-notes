$workspaceId = "<YOUR-WORKSPACE-ID>"

$query = @"
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
"@

$body = @{
    query = $query
} | ConvertTo-Json

$response = Invoke-AzRestMethod `
    -Method POST `
    -Path "https://api.loganalytics.azure.com/v1/workspaces/$workspaceId/query" `
    -Payload $body

$response.Content