WVDConnections
| where TimeGenerated > ago(1d)
| summarize DailyUsers = dcount(UserName)
| extend WeeklyProjection = DailyUsers * 5
| extend MonthlyProjection = DailyUsers * 22
| extend DailyActual = DailyUsers
| project DailyActual, WeeklyProjection, 

WVDConnections
| where TimeGenerated > ago(7d)
| extend DayName = format_datetime(TimeGenerated, "dddd")
| summarize UniqueUsers = dcount(UserName) by DayName
| order by case(
    DayName == "Monday", 1,
    DayName == "Tuesday", 2,
    DayName == "Wednesday", 3,
    DayName == "Thursday", 4,
    DayName == "Friday", 5,
    DayName == "Saturday", 6,
    DayName == "Sunday", 7,
    99
)