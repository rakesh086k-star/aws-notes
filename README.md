Get-SecureBootUEFI -Name db


Confirm-SecureBootUEFI



search *
| summarize Count=count() by $table
| sort by Count desc


union withsource=TableName *
| summarize Records=count() by TableName
| sort by Records desc








Total Active Session Hosts

Use Workbook Tile → Big Number
Heartbeat
| summarize Hosts=dcount(Computer)


2- Online vs Offline Hosts
Heartbeat
| summarize LastHeartbeat=max(TimeGenerated) by Computer
| extend Status=iff(LastHeartbeat > ago(10m),"Online","Offline")
| summarize Count=count() by Status


Query 3 - Active Users
WVDConnections
| where TimeGenerated > ago(24h)
| summarize ActiveUsers=dcount(UserName)





Query 4 - Session Success Rate
WVDConnections
| where TimeGenerated > ago(24h)
| summarize
TotalSessions=count(),
SuccessfulSessions=countif(State=="Connected")
| extend SuccessRate=round((SuccessfulSessions*100.0)/TotalSessions,2)



Query 7 - Top CPU Consumers

Perf
| where CounterName == "% Processor Time"
| summarize AvgCPU=avg(CounterValue) by Computer
| top 10 by AvgCPU desc




Query 8 - Memory Utilization
Perf
| where CounterName == "% Committed Bytes In Use"
| summarize AvgMemory=avg(CounterValue) by Computer
| top 10 by AvgMemory desc




Query 9 - Disk Space
Perf
| where CounterName == "% Free Space"
| summarize AvgFreeSpace=avg(CounterValue) by Computer
| top 10 by AvgFreeSpace asc


Visualization:
	•	Bar Chart

Color Rule:

🔴 Below 10%

🟡 10-20%

🟢 Above 20%


Query 10 - Top Connection Errors
WVDErrors
| summarize ErrorCount=count() by Code
| top 10 by ErrorCount desc

Visualization:
	•	Table



Query 11 - Failed Connections Trend
WVDErrors
| summarize Errors=count() by bin(TimeGenerated,1h)
| render timechart



Query 12 - Login Performance
WVDCheckpoints
| summarize AvgDuration=avg(DurationMs) by CheckpointName


Query 13 - Host Health Score
WVDAgentHealthStatus
| summarize Healthy=countif(Status=="Available"),
Unhealthy=countif(Status!="Available")




Host Availability Percentage
Heartbeat
| summarize LastHeartbeat=max(TimeGenerated) by Computer
| extend Status=iff(LastHeartbeat > ago(10m),"Online","Offline")
| summarize Online=countif(Status=="Online"),
            Offline=countif(Status=="Offline")
| extend Availability=round((Online*100.0)/(Online+Offline),2)

Visualization: Big Number





