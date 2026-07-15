Get-AppxPackage -AllUsers *Handwriting* | Remove-AppxPackage



Get-AppxProvisionedPackage -Online |
Where-Object DisplayName -like "*Handwriting*" |
Remove-AppxProvisionedPackage -Online



search *
| summarize Count=count() by $table
| sort by $table asc


union withsource=TableName *
| summarize Count=count() by TableName
| sort by TableName asc


Resources
| where type =~ "microsoft.compute/virtualMachines"
| where resourceGroup == "RG-AVD-WRR-E1-US"
| summarize Count=count() by PowerState=tostring(properties.extended.instanceView.powerState.code)


WVDConnections
| where TimeGenerated >= ago(24h)
| summarize Sessions = count() by GatewayRegion



WVDConnections
| where TimeGenerated >= ago(24h)
| extend Country = geo_info_from_ip_address(ClientIPAddress).country
| summarize Users = dcount(UserName) by Country
| order by Users desc

WVDConnections
| where TimeGenerated >= ago(24h)
| summarize TotalConnections = count() by GatewayRegion
| order by TotalConnections desc


WVDConnections
| where TimeGenerated >= ago(24h)
| project TimeGenerated, UserName, GatewayRegion, SessionHostName
| order by TimeGenerated desc



WVDConnections
| where TimeGenerated >= ago(24h)
| summarize arg_max(TimeGenerated, *) by UserName
| project TimeGenerated, UserName, GatewayRegion, SessionHostName
| order by TimeGenerated desc

WVDConnections
| where TimeGenerated >= ago(24h)
| summarize arg_max(TimeGenerated, *) by UserName
| project UserName, GatewayRegion, SessionHostName, RoundTripTimeInMs
| order by RoundTripTimeInMs desc





WVDConnectionNetworkData
| where TimeGenerated > ago(1d)
| join kind=inner (
    WVDConnections
    | where State == "Connected" and UserName != ""
    | summarize arg_max(TimeGenerated, *) by UserName
) on CorrelationId
| summarize
    AvgRTT = round(avg(EstRoundTripTimeInMs)),
    AvgBandwidth = round(avg(EstAvailableBandwidthKbps)),
    MaxRTT = max(EstRoundTripTimeInMs)
by UserName, GatewayRegion, SessionHostName
| order by AvgRTT desc



WVDConnectionNetworkData
| where TimeGenerated > ago(1d)
| summarize
    AvgRTT = round(avg(EstRoundTripTimeInMs)),
    AvgBandwidth = round(avg(EstAvailableBandwidthKbps))
by CorrelationId
| join kind=inner (
    WVDConnections
    | where State == "Connected"
    | summarize arg_max(TimeGenerated, *) by UserName
) on CorrelationId
| project UserName, GatewayRegion, SessionHostName, AvgRTT, AvgBandwidth



WVDConnectionNetworkData
| where TimeGenerated >= ago(1d)
| join kind=inner (
    WVDConnections
    | where State == "Connected" and UserName != ""
) on CorrelationId
| summarize
    AvgRTT = round(avg(EstRoundTripTimeInMs),0),
    AvgBandwidth = round(avg(EstAvailableBandwidthKBps),0),
    MaxRTT = max(EstRoundTripTimeInMs),
    MaxBandwidth = max(EstAvailableBandwidthKBps)
by UserName, GatewayRegion, SessionHostName
| order by AvgRTT desc




WVDConnectionNetworkData
| where TimeGenerated >= ago(1d)
| join kind=inner (
    WVDConnections
    | where State == "Connected" and UserName != ""
) on CorrelationId
| summarize
    ["Avg. RTT"] = strcat(round(avg(EstRoundTripTimeInMs),0), " ms"),
    ["Max. RTT"] = strcat(max(EstRoundTripTimeInMs), " ms"),
    ["P90 RTT"] = strcat(round(percentile(EstRoundTripTimeInMs,90),0), " ms"),
    ["Avg. Bandwidth"] = strcat(round(avg(EstAvailableBandwidthKBps)/1024.0,2), " MB/s"),
    ["Max. Bandwidth"] = strcat(round(max(EstAvailableBandwidthKBps)/1024.0,2), " MB/s"),
    ["P90 Bandwidth"] = strcat(round(percentile(EstAvailableBandwidthKBps,90)/1024.0,2), " MB/s")
by UserName
| order by todouble(split(['Avg. RTT'], " ")[0]) desc





WVDConnectionNetworkData
| where TimeGenerated >= ago(1d)
| join kind=inner (
    WVDConnections
    | where State == "Connected" and UserName != ""
    | project CorrelationId, UserName, GatewayRegion, SessionHostName
) on CorrelationId
| summarize
    ["Avg. RTT"] = strcat(round(avg(EstRoundTripTimeInMs),0), " ms"),
    ["Max. RTT"] = strcat(max(EstRoundTripTimeInMs), " ms"),
    ["Avg. Bandwidth"] = strcat(round(avg(EstAvailableBandwidthKBps)/1024.0,2), " MB/s"),
    ["Max. Bandwidth"] = strcat(round(max(EstAvailableBandwidthKBps)/1024.0,2), " MB/s")
by UserName, GatewayRegion, SessionHostName
| order by UserName asc




WVDConnections
| where TimeGenerated >= ago(1d)
| extend Geo = geo_info_from_ip_address(ClientIPAddress)
| project UserName, ClientIPAddress,
          Country = tostring(Geo.country),
          City = tostring(Geo.city)
| take 10

