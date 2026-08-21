WVDAgentHealthStatus
| where isnotempty(SessionHostName)
| summarize Records=count(), LastSeen=max(TimeGenerated) by SessionHostName
| order by SessionHostName asc


WVDAgentHealthStatus
| where isnotempty(SessionHostName)
| summarize LastSeen=max(TimeGenerated), Records=count() by SessionHostName
| order by LastSeen desc