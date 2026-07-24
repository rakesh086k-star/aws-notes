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




az vm list --resource-group "RG-AVD-LISS-IN-UK" --query "[].{Name:name,Size:hardwareProfile.vmSize}" --output table

az vm list -g "RG-AVD-LISS-IN-UK" --show-details --query "[].{VMName:name,ComputeSize:hardwareProfile.vmSize}" -o table


az account show --output table

az account list --all --output table


az account set --subscription "<Subscription Name या Subscription ID>"



Connect-AzAccount -Identity

$vnets = Get-AzVirtualNetwork

foreach ($vnet in $vnets)
{
    foreach ($subnet in $vnet.Subnets)
    {
        $usedIPs = ($subnet.IpConfigurations).Count

        Write-Output "VNet : $($vnet.Name)"
        Write-Output "Subnet : $($subnet.Name)"
        Write-Output "Address Prefix : $($subnet.AddressPrefix)"
        Write-Output "Used IP : $usedIPs"
    }
}




Connect-AzAccount -Identity

Write-Output "Login Successful"

Get-AzContext


Connect-AzAccount -Identity

Get-AzVirtualNetwork




let TotalUsers =
WVDConnections
| where TimeGenerated >= ago(1d)
| summarize dcount(UserName);

WVDConnections
| where TimeGenerated >= ago(1d)
| summarize arg_max(TimeGenerated, *) by UserName
| extend AccessMethod = case(
    ClientType contains "web", "Web Browser",
    ClientType contains "msrdc", "Windows App",
    ClientType contains "msrdcx", "Windows App",
    ClientType contains "android", "Windows App (Android)",
    ClientType contains "ios", "Windows App (iOS)",
    ClientType contains "mac", "Windows App (macOS)",
    "Other"
)
| summarize Users = dcount(UserName) by AccessMethod, ClientVersion
| extend Percentage = round((todouble(Users) / TotalUsers) * 100, 1)
| order by Users desc








WVDConnections
| where TimeGenerated >= ago(1d)
| summarize arg_max(TimeGenerated, *) by UserName
| extend AccessMethod = case(
    ClientType contains "web", "Web Browser",
    ClientType contains "msrdc" or ClientType contains "msrdcx", "Windows App",
    ClientType contains "android", "Windows App (Android)",
    ClientType contains "ios", "Windows App (iOS)",
    ClientType contains "mac", "Windows App (macOS)",
    "Other"
)
| summarize Users = count() by AccessMethod, ClientVersion
| order by AccessMethod asc, Users desc










let UserData =
WVDConnections
| where TimeGenerated >= ago(1d)
| summarize arg_max(TimeGenerated, *) by UserName
| extend AccessMethod = case(
    ClientType contains "web", "Web Browser",
    ClientType contains "msrdc" or ClientType contains "msrdcx", "Windows App",
    ClientType contains "android", "Windows App (Android)",
    ClientType contains "ios", "Windows App (iOS)",
    ClientType contains "mac", "Windows App (macOS)",
    "Other"
);

let TotalUsers = toscalar(
    UserData
    | summarize dcount(UserName)
);

UserData
| summarize Users = dcount(UserName) by AccessMethod, ClientVersion
| extend Percentage = strcat(round((todouble(Users) * 100.0) / TotalUsers, 1), "%")
| order by Users desc







Add-AppProvisionedAppxPackage -Online -PackagePath "C:\Temp\Application.msix" -SkipLicense


DISM /Online /Add-ProvisionedAppxPackage /PackagePath:"C:\Temp\Application.msix" /SkipLicense
















WVDConnections
| project UserName, SessionHostName, CorrelationId
| join kind=inner (
    WVDErrors
    | project CorrelationId, Code=tostring(Code), Message
) on CorrelationId
| summarize
    ErrorCount = count(),
    SampleMessage = any(Message)
    by SessionHostName, UserName, Code
| extend ErrorCategory = case(
    Code in ("3076","2055"), "🌐 Client Network",
    Code == "-2147467259", "❌ Session Host",
    Code == "516", "⚠️ AVD Connectivity",
    Code == "90", "🌐 Client Network",
    Code == "68", "🔌 Connection",
    Code == "27396", "🖥️ Session",
    Code == "50331694", "🌐 Client Network",
    "❓ Unknown"
)
| extend ErrorDescription = case(
    Code == "-2147467259", "Unexpected system error on the session host.",
    Code == "516", "Unable to connect to Azure Virtual Desktop service.",
    Code == "90", "Session host lost connection to the client.",
    Code == "68", "Connection terminated unexpectedly.",
    Code == "27396", "User session disconnected unexpectedly.",
    Code == "50331694", "Client network interruption detected.",
    Code in ("3076","2055"), "Client network issue detected.",
    SampleMessage
)
| extend RecommendedAction = case(
    Code == "-2147467259", "Check VM health, AVD Agent, FSLogix and Event Viewer.",
    Code == "516", "Verify Azure connectivity, DNS, Firewall and AVD service.",
    Code == "90", "Check client internet, VPN or Wi-Fi connection.",
    Code == "68", "Review Event Viewer and session logs.",
    Code == "27396", "Check session host health and user sign-in logs.",
    Code == "50331694", "Verify ISP, VPN and client network stability.",
    Code in ("3076","2055"), "Check client network connectivity.",
    "Investigate the Sample Message."
)
| project
    SessionHostName,
    UserName,
    ErrorCategory,
    Code,
    ErrorDescription,
    ErrorCount,
    RecommendedAction,
    SampleMessage
| order by ErrorCount desc
