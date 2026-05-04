WVDConnections
| where TimeGenerated > ago(1d)
| summarize DailyUsers = dcount(UserName)
| extend WeeklyProjection = DailyUsers * 5
| extend MonthlyProjection = DailyUsers * 22
| extend DailyActual = DailyUsers
| project DailyActual, WeeklyProjection, 

WVDConnections
| where TimeGenerated > ago(7d)
| extend DayNumber = dayofweek(TimeGenerated)
| extend DayName = case(
    DayNumber == 0d, "Sunday",
    DayNumber == 1d, "Monday",
    DayNumber == 2d, "Tuesday",
    DayNumber == 3d, "Wednesday",
    DayNumber == 4d, "Thursday",
    DayNumber == 5d, "Friday",
    DayNumber == 6d, "Saturday",
    "Unknown"
)
| summarize UniqueUsers = dcount(UserName) by DayName, DayNumber
| order by DayNumber asc