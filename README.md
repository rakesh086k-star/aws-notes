let TotalMachines =
    WVDAgentHealthStatus
    | summarize TotalMachines = dcount(SessionHostName);

let ActiveLast2Hours =
    WVDConnections
    | where TimeGenerated >= ago(2h)
    | summarize ActiveMachines = dcount(SessionHostName);

TotalMachines
| extend ActiveMachines = toscalar(ActiveLast2Hours)
| extend NonRunningMachines = TotalMachines - ActiveMachines
| project
    TotalMachines,
    ActiveMachines,
    NonRunningMachines