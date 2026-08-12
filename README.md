let SelectedUsers = dynamic([
    "Udit146824@exlservice.com",
    "Simrita160093@exlservice.com",
    "Akanksha170280@exlservice.com",
    "Kritika205648@exlservice.com",
    "Akanksh161389@exlservice.com",
    "Yashwant160794@exlservice.com"
]);

WVDConnections
| where TimeGenerated >= ago(7d)
| where UserName in~ (SelectedUsers)
| project
    ConnectionTime = TimeGenerated,
    UserName,
    SessionHostName,
    CorrelationId
| join kind=inner (
    WVDErrors
    | where TimeGenerated >= ago(7d)
    | project
        ErrorTime = TimeGenerated,
        CorrelationId,
        Category,
        Severity,
        Priority,
        HealthStatus,
        RootCause,
        Code,
        ErrorDescription,
        RecommendedAction
) on CorrelationId
| project
    ConnectionTime,
    ErrorTime,
    UserName,
    Category,
    Severity,
    Priority,
    HealthStatus,
    RootCause,
    Code,
    ErrorDescription,
    RecommendedAction
| order by ErrorTime desc