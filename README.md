let SelectedUsers = dynamic([
    "Udit146824@exlservice.com",
    "Simrita160093@exlservice.com",
    "Akanksha170280@exlservice.com",
    "Kritika205648@exlservice.com",
    "Akanksh161389@exlservice.com",
    "Yashwant160794@exlservice.com"
]);

WVDErrors
| where TimeGenerated >= ago(7d)
| where UserName in~ (SelectedUsers)
| where ActivityType == "Connection"
| project
    TimeGenerated,
    UserName,
    ActivityType,
    Source,
    ServiceError,
    Code,
    CodeSymbolic,
    Message,
    Operation,
    CorrelationId
| order by TimeGenerated desc