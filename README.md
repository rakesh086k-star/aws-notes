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
| order by LoginDate asc




