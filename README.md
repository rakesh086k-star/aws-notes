let CPUAlert =
Perf
| where TimeGenerated > ago(4h)
| where ObjectName in ("Processor", "Processor Information")
| where CounterName == "% Processor Time"
| where InstanceName == "_Total"
| summarize AvgCPU = avg(CounterValue) by Computer
| where AvgCPU > 80
| project Computer, Metric="CPU", Value=round(AvgCPU,2);

let MemoryAlert =
Perf
| where TimeGenerated > ago(4h)
| where ObjectName == "Memory"
| where CounterName == "% Committed Bytes In Use"
| summarize AvgMemory = avg(CounterValue) by Computer
| where AvgMemory > 80
| project Computer, Metric="Memory", Value=round(AvgMemory,2);

let DiskAlert =
Perf
| where TimeGenerated > ago(4h)
| where ObjectName == "LogicalDisk"
| where CounterName == "% Free Space"
| where InstanceName != "_Total"
| summarize AvgDiskFree = avg(CounterValue) by Computer, InstanceName
| where AvgDiskFree < 20
| project Computer, Metric=strcat("Disk (", InstanceName, ")"), Value=round(AvgDiskFree,2);

CPUAlert
| union MemoryAlert
| union DiskAlert
| order by Computer asc