WVDConnections
| summarize 
    DailyUsers = dcountif(UserName, TimeGenerated > ago(1d)),
    WeeklyUsers = dcountif(UserName, TimeGenerated > ago(7d)),
    MonthlyUsers = dcountif(UserName, TimeGenerated > ago(30d))