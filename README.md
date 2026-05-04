let CPUData = Perf
| where TimeGenerated > ago(30m)
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| summarize AvgCPU = avg(CounterValue) by Computer;

let MemData = Perf
| where TimeGenerated > ago(30m)
| where ObjectName == "Memory" and CounterName == "% Committed Bytes In Use"
| summarize AvgMem = avg(CounterValue) by Computer;

let HeartbeatData = Heartbeat
| where TimeGenerated > ago(30m)
| summarize LastSeen = max(TimeGenerated) by Computer;

// Use FULL OUTER JOIN (important fix)
CPUData
| join kind=fullouter MemData on Computer
| join kind=fullouter HeartbeatData on Computer
| extend AvgCPU = coalesce(AvgCPU, 0.0),
         AvgMem = coalesce(AvgMem, 0.0)
| extend HealthStatus = case(
    isnull(LastSeen) or LastSeen < ago(10m), "Critical",
    AvgCPU > 85 or AvgMem > 85, "Critical",
    AvgCPU > 70 or AvgMem > 70, "Warning",
    "Healthy"
)
| summarize Total = count() by HealthStatus