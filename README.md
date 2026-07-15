hu Ji Get-AppxPackage -AllUsers *Handwriting* | Remove-AppxPackage



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





WVDConnections
| where TimeGenerated >= ago(24h)
| extend Geo = geo_info_from_ip_address(ClientIPAddress)
| project
    UserName,
    ClientIPAddress,
    Country = tostring(Geo.country),
    City = tostring(Geo.city)
| take 20



WVDConnections
| where TimeGenerated >= ago(24h)
| project UserName, ClientIPAddress
| take 20




WVDConnections
| where TimeGenerated >= ago(24h)
| summarize arg_max(TimeGenerated, *) by UserName
| extend Geo = geo_info_from_ip_address(ClientIPAddress)
| project
    UserName,
    Country = tostring(Geo.country),
    City = tostring(Geo.city),
    ClientIPAddress
| order by UserName asc







WVDConnectionNetworkData
| where TimeGenerated >= ago(1d)
| join kind=inner (
    WVDConnections
    | where State == "Connected" and UserName != ""
    | summarize arg_max(TimeGenerated, *) by UserName
    | extend Geo = geo_info_from_ip_address(ClientIPAddress)
    | extend Country = tostring(Geo.country),
             City = tostring(Geo.city)
    | project
        CorrelationId,
        UserName,
        SessionHostName,
        GatewayRegion,
        Country,
        City,
        ClientIPAddress,
        ClientType
) on CorrelationId
| summarize
    ["Avg RTT"] = strcat(round(avg(EstRoundTripTimeInMs),0), " ms"),
    ["Max RTT"] = strcat(round(max(EstRoundTripTimeInMs),0), " ms"),
    ["Avg Bandwidth"] = strcat(round(avg(EstAvailableBandwidthKBps)/1024.0,2), " MB/s"),
    ["Max Bandwidth"] = strcat(round(max(EstAvailableBandwidthKBps)/1024.0,2), " MB/s")
by
    UserName,
    SessionHostName,
    GatewayRegion,
    Country,
    City,
    ClientIPAddress,
    ClientType
| order by UserName asc




WVDConnectionNetworkData
| where TimeGenerated >= ago(1d)
| join kind=inner (
    WVDConnections
    | where State == "Connected" and UserName != ""
    | summarize arg_max(TimeGenerated, *) by UserName
    | extend Geo = geo_info_from_ip_address(ClientIPAddress)
    | extend Country = tostring(Geo.country),
             City = tostring(Geo.city)
    | extend AccessMethod = case(
        ClientType contains "web", "Web Browser",
        ClientType contains "msrdcx64", "Windows App",
        ClientType contains "msrdc", "Remote Desktop",
        ClientType contains "android", "Android App",
        ClientType contains "ios", "iOS App",
        ClientType contains "mac", "macOS App",
        ClientType
    )
    | project
        CorrelationId,
        UserName,
        AccessMethod,
        ClientVersion,
        GatewayRegion,
        Country,
        City,
        ClientIPAddress
) on CorrelationId
| summarize
    ["Avg RTT"] = strcat(round(avg(EstRoundTripTimeInMs),0), " ms"),
    ["Max RTT"] = strcat(round(max(EstRoundTripTimeInMs),0), " ms"),
    ["Avg Bandwidth"] = strcat(round(avg(EstAvailableBandwidthKBps)/1024.0,2), " MB/s"),
    ["Max Bandwidth"] = strcat(round(max(EstAvailableBandwidthKBps)/1024.0,2), " MB/s")
by
    UserName,
    AccessMethod,
    ClientVersion,
    GatewayRegion,
    Country,
    City,
    ClientIPAddress
| order by UserName asc



WVDConnections
| where TimeGenerated >= ago(7d)
| summarize Users = dcount(UserName) by ClientType, ClientVersion
| order by ClientType asc, ClientVersion desc



WVDConnections
| where TimeGenerated >= ago(7d)
| summarize Users = dcount(UserName) by ClientType, ClientVersion
| order by ClientType asc, ClientVersion desc



WVDConnections
| where TimeGenerated >= ago(7d)
| summarize arg_max(TimeGenerated, *) by UserName
| extend AccessMethod = case(
    ClientType contains "web", "Web Browser",
    ClientType contains "msrdc", "Windows App",
    ClientType contains "msrdcx", "Windows App",
    ClientType contains "android", "Windows App (Android)",
    ClientType contains "ios", "Windows App (iOS)",
    ClientType contains "mac", "Windows App (macOS)",
    ClientType
)
| project
    UserName,
    AccessMethod,
    ClientVersion,
    ClientOS,
    TimeGenerated
| order by UserName asc








WVDConnectionNetworkData
| where TimeGenerated >= ago(1d)
| join kind=inner (
    WVDConnections
    | where State == "Connected" and UserName != ""
    | summarize arg_max(TimeGenerated, *) by UserName
    | extend Geo = geo_info_from_ip_address(ClientIPAddress)
    | extend Country = tostring(Geo.country),
             City = tostring(Geo.city)
    | extend AccessMethod = case(
        ClientType contains "web", "Web Browser",
        ClientType contains "msrdc", "Windows App",
        ClientType contains "msrdcx", "Windows App",
        ClientType contains "android", "Windows App (Android)",
        ClientType contains "ios", "Windows App (iOS)",
        ClientType contains "mac", "Windows App (macOS)",
        "Other"
    )
    | project
        CorrelationId,
        UserName,
        AccessMethod,
        ClientVersion,
        GatewayRegion,
        Country,
        City,
        ClientIPAddress
) on CorrelationId
| summarize
    AvgRTT = round(avg(EstRoundTripTimeInMs),0),
    MaxRTT = round(max(EstRoundTripTimeInMs),0),
    AvgBandwidth = round(avg(EstAvailableBandwidthKBps)/1024.0,2),
    MaxBandwidth = round(max(EstAvailableBandwidthKBps)/1024.0,2)
by
    UserName,
    AccessMethod,
    ClientVersion,
    GatewayRegion,
    Country,
    City,
    ClientIPAddress
| order by AvgRTT desc


