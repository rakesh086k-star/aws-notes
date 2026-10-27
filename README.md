
WVDConnections
| where TimeGenerated >= ago(6d)
| where isnotempty(UserName)
| where isnotempty(SessionHostName)
| extend LoginDate = startofday(TimeGenerated)
| extend DayName = case(
    dayofweek(LoginDate) == 0d, "Sunday",
    dayofweek(LoginDate) == 1d, "Monday",
    dayofweek(LoginDate) == 2d, "Tuesday",
    dayofweek(LoginDate) == 3d, "Wednesday",
    dayofweek(LoginDate) == 4d, "Thursday",
    dayofweek(LoginDate) == 5d, "Friday",
    dayofweek(LoginDate) == 6d, "Saturday",
    "Unknown"
)
| extend ComputerName = tostring(SessionHostName),
         UserName = tostring(UserName)
| summarize UniqueUsers = dcount(UserName)
    by LoginDate, DayName, ComputerName
| project
    DayName,
    LoginDate,
    ComputerName,
    UniqueUsers
| order by LoginDate desc, ComputerName asc
