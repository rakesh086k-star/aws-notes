let Base = Heartbeat
| summarize LastSeen = max(TimeGenerated) by Computer;

let Total = toscalar(Base | summarize count());

Base
| extend Status = iff(LastSeen > ago(10m), "Active", "Inactive")
| summarize Count = count() by Status
| extend TotalSystems = Total


Heartbeat
| summarize LastSeen = max(TimeGenerated) by Computer
| where LastSeen < ago(10m)