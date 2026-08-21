WVDAgentHealthStatus
| where isnotempty(SessionHostName)
| summarize TotalMachine = dcount(SessionHostName)