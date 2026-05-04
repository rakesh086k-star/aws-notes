let Base = Heartbeat
| summarize LastSeen = max(TimeGenerated) by Computer;

let Total = toscalar(Base | summarize count());

let StatusData =
Base
| extend Status = iff(LastSeen > ago(10m), "Active", "Inactive")
| summarize Count = count() by Status;

datatable(Status:string)
[
"Active",
"Inactive"
]
| join kind=leftouter StatusData on Status
| extend Count = coalesce(Count, 0),
         TotalSystems = Total