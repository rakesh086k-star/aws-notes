WVDConnections
| where TimeGenerated > ago(24h)
| where State == "Connected"
| summarize ActiveUsers = dcount(UserName)


WVDConnections
| where TimeGenerated > ago(24h)
| summarize ActiveUsers = dcount(UserName)


WVDConnections
| where TimeGenerated > ago(24h)
| where State == "Disconnected"
| summarize DisconnectedSessions = count()

WVDCheckpoints
| where TimeGenerated > ago(24h)
| where Name contains "Disconnected"
| summarize DisconnectedSessions = count()

WVDAgentHealthStatus
| where TimeGenerated > ago(15m)
| where SessionState == "Idle"
| summarize IdleSessions = count()



WVDAgentHealthStatus
| where TimeGenerated > ago(15m)
| summarize IdleSessions = dcount(SessionHostName)

let Active =
WVDConnections
| where TimeGenerated > ago(24h)
| where State == "Connected"
| summarize Value=dcount(UserName)
| extend Metric="Active Users";

let Disconnected =
WVDConnections
| where TimeGenerated > ago(24h)
| where State == "Disconnected"
| summarize Value=count()
| extend Metric="Disconnected";

union Active, Disconnected





