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
| extend AvgDiskUsed = 100 - AvgDiskFree;

CPU
| join kind=fullouter Memory on Computer
| join kind=fullouter Disk on Computer
| project Computer,
    AvgCPU = round(AvgCPU,2),
    AvgMemory = round(AvgMemory,2),
    AvgDisk = round(AvgDiskUsed,2)
| extend CPUStatus = case(
        AvgCPU > 80, "🔴 Critical",
        AvgCPU > 60, "🟡 Warning",
        "🟢 Healthy"
    )
| extend MemStatus = case(
        AvgMemory > 80, "🔴 Critical",
        AvgMemory > 60, "🟡 Warning",
        "🟢 Healthy"
    )
| extend DiskStatus = case(
        AvgDisk > 80, "🔴 Critical",
        AvgDisk > 60, "🟡 Warning",
        "🟢 Healthy"
    )
| order by AvgCPU desc