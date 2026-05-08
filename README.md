WVDConnections
| project UserName, CorrelationId, SessionHostName
| join kind=inner (
    WVDErrors
    | project CorrelationId, Code, Message, ServiceError
) on CorrelationId
| summarize
    ErrorCount = count(),
    ErrorCodes = make_set(Code, 3),
    ErrorMessages = make_set(Message, 3)
    by SessionHostName, UserName
| extend Severity = case(
    ErrorCount >= 10, "Critical 🔴",
    ErrorCount >= 5, "High 🟠",
    ErrorCount >= 2, "Medium 🟡",
    "Low 🟢"

WVDConnections
| project UserName, CorrelationId, SessionHostName
| join kind=inner (
    WVDErrors
    | project CorrelationId
) on CorrelationId
| summarize ErrorCount=count() by SessionHostName, UserName
| order by ErrorCount desc

)
| order by ErrorCount desc




WVDConnections
| project UserName, SessionHostName, CorrelationId
| join kind=inner (
    WVDErrors
    | project CorrelationId, Code, Message
) on CorrelationId
| summarize
    ErrorCount = count(),
    SampleMessage = any(Message)
    by SessionHostName, UserName, Code
| extend ErrorSource = case(
    Code == "3076", "Client Network Issue 🌐",
    Code == "2055", "Session Connectivity Issue ⚠️",
    Code == "-2147467259", "Unknown/System Error ❌",
    "Other Error"
)
| order by ErrorCount desc



WVDConnections
| project UserName, SessionHostName, CorrelationId
| join kind=inner (
    WVDErrors
    | project CorrelationId, Code, Message
) on CorrelationId
| summarize
    ErrorCount = count(),
    SampleMessage = any(Message)
    by SessionHostName, UserName, Code
| extend ErrorSource = case(
    Code in ("3076", "2055"), "Client Network Issue 🌐",
    Code == "-2147467259", "System Error ❌",
    Code == "516", "AVD Connectivity Issue ⚠️",
    "Other Error"
)
| project
    SessionHostName,
    UserName,
    ErrorSource,
    Code,
    ErrorCount,
    SampleMessage
| order by ErrorCount desc





Perf
| where TimeGenerated > ago(1d)
| where (ObjectName == "Logical Disk" or ObjectName == "LogicalDisk")
and CounterName contains "% Free Space"
and InstanceName != "_Total"
and InstanceName != "HarddiskVolume1"
| extend AVDHostName = Computer
| extend FreeSpace = CounterValue
| extend DriveName = InstanceName
| extend FreeSpaceGB = FreeSpace
| extend SpaceStatus = case(
    FreeSpaceGB < 5, "Critical",
    FreeSpaceGB < 10, "Warning",
    FreeSpaceGB < 30, "Normal",
    "Medium"
)
| summarize arg_max(TimeGenerated, *) by AVDHostName
| project AVDHostName, FreeSpaceGB, SpaceStatus, TimeGenerated
| order by FreeSpaceGB desc




