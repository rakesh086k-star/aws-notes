let TimeWindow = 2h;

let TotalMachines =
    WVDConnections
    | summarize TotalMachines = dcount(SessionHostName);

let RunningMachines =
    WVDConnections
    | where TimeGenerated >= ago(TimeWindow)
    | summarize RunningMachines = dcount(SessionHostName);

TotalMachines
| join kind=fullouter RunningMachines on $left.TotalMachines == $right.RunningMachines
| extend TotalMachines = coalesce(TotalMachines, 0),
         RunningMachines = coalesce(RunningMachines, 0)
| extend NonRunningMachines = TotalMachines - RunningMachines
| project
    TotalMachines,
    RunningMachines,
    NonRunningMachines