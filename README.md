let CPUData = Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| summarize AvgCPU = avg(CounterValue) by Computer;

let MemData = Perf
| where ObjectName == "Memory" and CounterName == "% Committed Bytes In Use"
| summarize AvgMem = avg(CounterValue) by Computer;

let HeartbeatData = Heartbeat
| summarize LastSeen = max(TimeGenerated) by Computer;

CPUData
| join kind=inner MemData on Computer
| join kind=inner HeartbeatData on Computer
| extend HealthStatus = case(
    LastSeen < ago(5m), "Critical",        // Machine is not reporting (down)
    AvgCPU > 85 or AvgMem > 85, "Critical",
    AvgCPU > 70 or AvgMem > 70, "Warning",
    "Healthy"
)
| summarize Total = count() by HealthStatus