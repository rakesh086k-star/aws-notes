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
| summarize AvgDiskFree = avg(CounterValue) by Computer
| extend AvgDisk = 100 - AvgDiskFree;

CPU
| join kind=fullouter Memory on Computer
| join kind=fullouter Disk on Computer
| extend CPUStatus = case(AvgCPU > 80, "Critical", AvgCPU > 60, "Warning", "Healthy")
| extend MemStatus = case(AvgMemory > 80, "Critical", AvgMemory > 60, "Warning", "Healthy")
| extend DiskStatus = case(AvgDisk > 80, "Critical", AvgDisk > 60, "Warning", "Healthy")
| extend OverallStatus = case(
    CPUStatus == "Critical" or MemStatus == "Critical" or DiskStatus == "Critical", "Critical",
    CPUStatus == "Warning" or MemStatus == "Warning" or DiskStatus == "Warning", "Warning",
    "Healthy"
)
| extend IssueArea = case(
    CPUStatus != "Healthy", "CPU Issue",
    MemStatus != "Healthy", "Memory Issue",
    DiskStatus != "Healthy", "Disk Issue",
    "Normal"
)
| project Computer, AvgCPU, AvgMemory, AvgDisk, OverallStatus, IssueArea
| order by OverallStatus desc