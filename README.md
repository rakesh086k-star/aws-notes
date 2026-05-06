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
| extend DiskUsed = 100 - AvgDiskFree;

CPU
| join kind=fullouter Memory on Computer
| join kind=fullouter Disk on Computer
| extend
    CPU_Used = round(AvgCPU, 0),
    Memory_Used = round(AvgMemory, 0),
    Disk_Used = round(DiskUsed, 0),
    Disk_Available = round(AvgDiskFree, 0)
| extend CPU_Display = strcat(CPU_Used, "%")
| extend Memory_Display = strcat(Memory_Used, "%")
| extend DiskUsed_Display = strcat(Disk_Used, "%")
| extend DiskAvail_Display = strcat(Disk_Available, "%")
| extend OverallStatus = case(
    CPU_Used > 80 or Memory_Used > 80 or Disk_Used > 80, "Critical",
    CPU_Used > 60 or Memory_Used > 60 or Disk_Used > 60, "Warning",
    "Healthy"
)
| project
    Computer,
    CPU_Display,
    Memory_Display,
    DiskUsed_Display,
    DiskAvail_Display,
    OverallStatus
| order by CPU_Used desc