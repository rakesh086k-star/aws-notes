WVDConnections
| where TimeGenerated >= ago(30d)
| summarize LastLogin=max(TimeGenerated) by UserName
| extend DaysSinceLastLogin = datetime_diff("day", now(), LastLogin)
| project UserName, LastLogin, DaysSinceLastLogin
| order by DaysSinceLastLogin asc