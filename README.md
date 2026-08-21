WVDAgentHealthStatus
| where isnotempty(SessionHostName)
| summarize Records = count(),
            UniqueMachines = dcount(SessionHostName)