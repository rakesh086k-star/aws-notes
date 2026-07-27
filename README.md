let LastLogin =
WVDConnections
| summarize LastLogin = max(TimeGenerated) by UserName;

LastLogin
| extend DaysSinceLastLogin = datetime_diff('day', now(), LastLogin) * -1
| extend LoginStatus = case(
    DaysSinceLastLogin == 0, "Today",
    DaysSinceLastLogin == 1, "Yesterday",
    DaysSinceLastLogin <= 7, "Last 7 Days",
    DaysSinceLastLogin <= 30, "Last 30 Days",
    "Inactive"
)
| extend Attention = case(
    DaysSinceLastLogin == 0, "Active",
    DaysSinceLastLogin <= 7, "Normal",
    DaysSinceLastLogin <= 30, "Warning",
    "Critical"
)
| extend UserActivity = case(
    DaysSinceLastLogin == 0, "Frequently Using AVD",
    DaysSinceLastLogin <= 7, "Regular User",
    DaysSinceLastLogin <= 30, "Occasional User",
    "Inactive User"
)
| project
    UserName,
    LastLogin,
    DaysSinceLastLogin,
    LoginStatus,
    Attention,
    UserActivity
| order by LastLogin desc