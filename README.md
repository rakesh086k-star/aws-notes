let CPU =
Perf
| where TimeGenerated > ago(4h)
| where CounterName == "% Processor Time"
| summarize AvgCPU = avg(CounterValue) by Computer;

let Memory =
Perf
| where TimeGenerated > ago(4h)
| where CounterName == "% Committed Bytes In Use"
| summarize AvgMemory = avg(CounterValue) by Computer;

let Disk =
Perf
| where TimeGenerated > ago(4h)
| where CounterName == "% Free Space"
| summarize AvgDiskFree = avg(CounterValue) by Computer;

CPU
| join kind=fullouter Memory on Computer
| join kind=fullouter Disk on Computer
| extend
    AvgCPU = round(AvgCPU, 0),
    AvgMemory = round(AvgMemory, 0),
    AvgDiskAvailable = round(AvgDiskFree, 0)
| extend CPUStatus = case(AvgCPU > 80, "Critical", AvgCPU > 60, "Warning", "Healthy")
| extend MemStatus = case(AvgMemory > 80, "Critical", AvgMemory > 60, "Warning", "Healthy")
| extend DiskStatus = case(AvgDiskAvailable < 20, "Critical", AvgDiskAvailable < 40, "Warning", "Healthy")
| extend OverallStatus = case(
    CPUStatus == "Critical" or MemStatus == "Critical" or DiskStatus == "Critical", "Critical",
    CPUStatus == "Warning" or MemStatus == "Warning" or DiskStatus == "Warning", "Warning",
    "Healthy"
)
| project
    Computer,
    AvgCPU,
    AvgMemory,
    AvgDiskAvailable,
    OverallStatus,
    CPUStatus,
    MemStatus,
    DiskStatus
| order by AvgCPU desc