let SelectedUsers = dynamic([
    "user1@company.com",
    "user2@company.com",
    "user3@company.com",
    "user4@company.com",
    "user5@company.com",
    "user6@company.com"
]);

let UserSessions =
WVDConnections
| where TimeGenerated >=