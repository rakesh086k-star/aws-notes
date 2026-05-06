let CPU =
Perf
| where TimeGenerated > ago(4h)
| where CounterName == "% Processor Time"
| summarize CPU_Used = avg(CounterValue) by Computer;

let Memory =
Perf
| where TimeGenerated > ago(4h)
| where CounterName == "% Committed Bytes In Use"
| summarize Memory_Used = avg(CounterValue) by Computer;

let Disk =
Perf
| where TimeGenerated > ago(4h)
| where CounterName == "% Free Space"
| summarize Disk_Free = avg(CounterValue) by Computer;

CPU
| join kind=fullouter Memory on Computer
| join kind=fullouter Disk on Computer
| extend
    CPU_Used = round(CPU_Used, 0),
    Memory_Used = round(Memory_Used, 0),
    Disk_Free = round(Disk_Free, 0),
    Disk_Used = 100 - Disk_Free
| extend Status = case(
    CPU_Used > 80 or Memory_Used > 80 or Disk_Used > 80, "Critical",
    CPU_Used > 60 or Memory_Used > 60 or Disk_Used > 60, "Warning",
    "Healthy"
)
| project
    Computer,
    CPU_Used,
    Memory_Used,
    Disk_Used,
    Disk_Free,
    Status
| order by CPU_Used desc