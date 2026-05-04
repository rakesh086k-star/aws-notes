WVDConnections
| where State == "Failed"
| summarize ErrorCount = count() by SessionHostName, ClientType
| order by ErrorCount desc