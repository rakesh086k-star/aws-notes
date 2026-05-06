let CPU =
Perf
| where TimeGenerated > ago(4h)
| where CounterName == "% Processor Time"
| summarize AvgCPU = avg(CounterValue) by Computer;

let Memory =
Perf
| where TimeGenerated > ago(4h)
| where CounterName == "% Committed Bytes In Use"
| summarize AvgMemoryUsed = avg(CounterValue) by Computer;

let Disk =
Perf
| where TimeGenerated > ago(4h)
| where CounterName == "% Free Space"
| summarize AvgDiskFree = avg(CounterValue) by Computer
| extend AvgDiskUsed = 100 - AvgDiskFree;

CPU
| join kind=fullouter Memory on Computer
| join kind=fullouter Disk on Computer
| extend
    CPU_Used = round(AvgCPU, 0),
    Memory_Used = round(AvgMemoryUsed, 0),
    Disk_Used = round(AvgDiskUsed, 0)
| extend CPUStatus = case(CPU_Used > 80, "Critical", CPU_Used > 60, "Warning", "Healthy")
| extend MemStatus = case(Memory_Used > 80, "Critical", Memory_Used > 60, "Warning", "Healthy")
| extend DiskStatus = case(Disk_Used > 80, "Critical", Disk_Used > 60, "Warning", "Healthy")
| extend OverallStatus = case(
    CPUStatus == "Critical" or MemStatus == "Critical" or DiskStatus == "Critical", "Critical",
    CPUStatus == "Warning" or MemStatus == "Warning" or DiskStatus == "Warning", "Warning",
    "Healthy"
)
| project
    Computer,
    CPU_Used,
    Memory_Used,
    Disk_Used,
    CPUStatus,
    MemStatus,
    DiskStatus,
    OverallStatus
| order by CPU_Used desc