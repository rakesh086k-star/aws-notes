let SelectedUsers = dynamic([
    "user1@company.com",
    "user2@company.com",
    "user3@company.com",
    "user4@company.com",
    "user5@company.com",
    "user6@company.com"
]);

WVDConnections
| where TimeGenerated >= ago(7d)
| where UserName in~ (SelectedUsers)
| project
    TimeGenerated,
    UserName,
    State,
    Source,
    ClientType,
    ClientVersion,
    ConnectionType,
    CorrelationId,
    *
| take 100
| order by TimeGenerated desc