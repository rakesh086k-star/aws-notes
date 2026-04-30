Perf
| where ObjectName == "Processor"
| where CounterName == "% Processor Time"
| where InstanceName == "_Total"
| summarize CPU_Usage = avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
| sort by TimeGenerated desc


Perf
| where ObjectName == "Memory"
| where CounterName == "% Committed Bytes In Use"
| summarize Memory_Usage = avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
| sort by TimeGenerated desc

WVDConnections
| where State == "Connected"
| extend LoginTime = datetime_diff('second', TimeGenerated, PredecessorConnectionTime)
| summarize AvgLoginTime = avg(LoginTime) by Computer, bin(TimeGenerated, 5m)
| sort by AvgLoginTime desc

Perf
| where ObjectName == "LogicalDisk"
| where CounterName == "Free Megabytes"
| extend FreeGB = CounterValue / 1024
| summarize arg_max(TimeGenerated, *) by Computer, InstanceName
| project Computer, Drive=InstanceName, FreeGB

Event
| where Source == "FSLogix"
| where EventLevelName in ("Error","Warning")
| summarize Count = count() by Computer, EventLevelName, bin(TimeGenerated, 10m)

Perf
| where CounterName == "Free Megabytes"
| extend FreeGB = CounterValue / 1024
| extend Status = case(
    FreeGB < 5, "🔴 Critical",
    FreeGB < 10, "🟠 Warning",
    FreeGB < 20, "🟡 Low",
    "🟢 Healthy"
)
| summarize arg_max(TimeGenerated, *) by Computer
| project Computer, FreeGB, Status

WVDConnections
| summarize Sessions = count() by Computer
| join kind=leftouter (
    Event
    | where Source == "FSLogix"
    | summarize Errors = count() by Computer
) on Computer




