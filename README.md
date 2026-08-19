Perf
| where TimeGenerated > ago(12h)
| summarize TotalMachines = dcount(Computer)

Perf
| where TimeGenerated > ago(15m)
| summarize RunningMachines = dcount(Computer)
