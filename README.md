Resources
| where type =~ "microsoft.desktopvirtualization/hostpools"
| project
    subscriptionId,
    resourceGroup,
    name,
    type,
    location,
    id
| order by subscriptionId, name





let StartTime = datetime(2026-09-16 00:00:00);
let EndTime   = datetime(2026-09-17 23:59:59);

let Connections =
WVDConnections
| where TimeGenerated between (StartTime .. EndTime)
| summarize
    ConnectionAttempts = count(),
    SuccessfulConnections = countif(State == "Connected"),
    LastConnection = max(TimeGenerated),
    SessionHost = any(SessionHostName),
    ClientIP = any(ClientSideIPAddress),
    ClientOS = any(ClientOS),
    ClientType = any(ClientType),
    ClientVersion = any(ClientVersion)
    by UserName;

let Errors =
WVDErrors
| where TimeGenerated between (StartTime .. EndTime)
| summarize
    ErrorCount = count(),
    ErrorCodes = make_set(CodeSymbolic, 10),
    LastError = max(TimeGenerated),
    ErrorMessage = any(Message)
    by UserName;

Connections
| join kind=fullouter Errors on UserName
| extend
    ConnectionAttempts = coalesce(ConnectionAttempts, 0),
    SuccessfulConnections = coalesce(SuccessfulConnections, 0),
    ErrorCount = coalesce(ErrorCount, 0)
| extend
    Status = case(
        ErrorCount > 0, "🔴 ERROR",
        ConnectionAttempts == 0, "🟠 NO AVD CONNECTION LOG",
        SuccessfulConnections > 0, "🟢 CONNECTED",
        "🟡 CHECK"
    )
| project
    Status,
    UserName,
    ConnectionAttempts,
    SuccessfulConnections,
    ErrorCount,
    ErrorCodes,
    ErrorMessage,
    LastError,
    LastConnection,
    SessionHost,
    ClientIP,
    ClientOS,
    ClientType,
    ClientVersion
| order by
    case(
        Status == "🔴 ERROR", 1,
        Status == "🟠 NO AVD CONNECTION LOG", 2,
        Status == "🟡 CHECK", 3,
        4
    ) asc,
    UserName asc
