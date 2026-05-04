WVDConnections
| where TimeGenerated > ago(1d)
| summarize DailyUsers = dcount(UserName)
| extend WeeklyProjection = DailyUsers * 5
| extend MonthlyProjection = DailyUsers * 22
| extend DailyActual = DailyUsers
| project DailyActual, WeeklyProjection, MonthlyProjection


WVDConnections
| where TimeGenerated > ago(7d)
| extend DayName = format_datetime(TimeGenerated, "dddd")
| summarize UniqueUsers = dcount(UserName) by DayName
| order by case(DayName,
    "Monday", 1,
    "Tuesday", 2,
    "Wednesday", 3,
    "Thursday", 4,
    "Friday", 5,
    "Saturday", 6,
    "Sunday", 7,
    8)