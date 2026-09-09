let LastLogin =
WVDConnections
| summarize arg_max(TimeGenerated, *) by UserName
| extend
    ComputerName = tostring(SessionHostName),
    LastLogin = TimeGenerated;

LastLogin
| extend DaysSinceLastLogin = datetime_diff("day", LastLogin, now())
| extend LoginStatus = case(
    DaysSinceLastLogin == 0, "Today",
    DaysSinceLastLogin == 1, "Yesterday",
    DaysSinceLastLogin == 2, "2 Days Ago",
    DaysSinceLastLogin == 3, "3 Days Ago",
    DaysSinceLastLogin == 4, "4 Days Ago",
    DaysSinceLastLogin == 5, "5 Days Ago",
    DaysSinceLastLogin == 6, "6 Days Ago",
    DaysSinceLastLogin == 7, "7 Days Ago",
    DaysSinceLastLogin <= 14, strcat(tostring(DaysSinceLastLogin), " Days Ago"),
    DaysSinceLastLogin <= 30, strcat(tostring(DaysSinceLastLogin), " Days Ago"),
    strcat(tostring(DaysSinceLastLogin), " Days Ago")
)
| project
    UserName,
    ComputerName,
    LastLogin,
    DaysSinceLastLogin,
    LoginStatus
| order by LastLogin desc