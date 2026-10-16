
let StartTime = datetime(2026-10-12 00:00:00);
let EndTime = datetime(2026-10-15 23:59:59);
let TargetUser = "Takshu255467@exlservice.com";
let UserHosts =
    WVDConnections
    | where TimeGenerated between (StartTime .. EndTime)
    | where UserName =~ TargetUser
    | where isnotempty(SessionHostName)
    | distinct SessionHostName;
Perf
| where TimeGenerated between (StartTime .. EndTime)
| where Computer in~ (UserHosts)
| where ObjectName contains "Processor"
    or CounterName contains "Processor"
| summarize
    Records = count(),
    LatestRecord = max(TimeGenerated),
    SampleCPUValue = max(CounterValue)
    by Computer, ObjectName, CounterName, InstanceName
| order by Computer asc
