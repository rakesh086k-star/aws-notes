WVDConnectionNetworkData
| where TimeGenerated >= ago(12h)
| join kind=inner (
    WVDConnections
    | where State == "Connected" and UserName != ""
    | summarize arg_max(TimeGenerated, *) by UserName
    | extend Geo = geo_info_from_ip_address(ClientIPAddress)
    | extend
        Country = tostring(Geo.country),
        City = tostring(Geo.city),
        ComputerName = tostring(SessionHostName)
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
        ComputerName,
        ClientVersion,
        GatewayRegion,
        Country,
        City,
        ClientIPAddress,
        AccessMethod
) on CorrelationId
| summarize
    AvgRTT = round(avg(EstRoundTripTimeInMs), 0),
    MaxRTT = round(max(EstRoundTripTimeInMs), 0),
    AvgBandwidth = round(avg(EstAvailableBandwidthKBps) / 1024.0, 2),
    MaxBandwidth = round(max(EstAvailableBandwidthKBps) / 1024.0, 2)
    by
        UserName,
        ComputerName,
        Country,
        City,
        GatewayRegion,
        ClientIPAddress,
        AccessMethod,
        ClientVersion
| order by AvgRTT desc