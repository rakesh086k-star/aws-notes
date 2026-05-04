WVDConnections
| where TimeGenerated > ago(1d)
| summarize DailyUsers = dcount(UserName)
| extend WeeklyProjection = DailyUsers * 5
| extend MonthlyProjection = DailyUsers * 22
| extend DailyActual = DailyUsers
| project DailyActual, WeeklyProjection, 

WVDConnections
| where TimeGenerated > ago(7d)
| extend LoginDate = startofday(TimeGenerated)
| extend DayName = format_datetime(TimeGenerated, "dddd")
| summarize UniqueUsers = dcount(UserName) by LoginDate, DayName
| order by LoginDate 
WVDConnections
| where TimeGenerated > ago(7d)
| extend LoginDate = startofday(TimeGenerated)
| extend DayName = format_datetime(LoginDate, "dddd")
| summarize UniqueUsers = dcount(UserName) by LoginDate, DayName
| order by LoginDate 


WVDConnections
| where TimeGenerated > ago(7d)
| extend LoginDate = startofday(TimeGenerated)
| extend DayNum = dayofweek(LoginDate)
| extend DayName = case(
    DayNum == 0d, "Sunday",
    DayNum == 1d, "Monday",
    DayNum == 2d, "Tuesday",
    DayNum == 3d, "Wednesday",
    DayNum == 4d, "Thursday",
    DayNum == 5d, "Friday",
    DayNum == 6d, "Saturday",
    "Unknown"
)
| summarize UniqueUsers = dcount(UserName) by LoginDate, DayName
| order by LoginDate 

WVDConnections
| where TimeGenerated > ago(7d)
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
| summarize UniqueUsers = dcount(UserName) by LoginDate, DayName
| project LoginDate, DayName, UniqueUsers
| order by LoginDate asc




