let CPUAlert =
Perf
| where TimeGenerated > ago(4h)
| where CounterName == "% Processor Time"
| summarize AvgCPU = avg(CounterValue) by Computer
| extend Status = case(AvgCPU > 80, "Critical", AvgCPU > 60, "Warning", "Healthy")
| project Computer, Metric="CPU", Value=round(AvgCPU,2), Status;

let MemoryAlert =
Perf
| where TimeGenerated > ago(4h)
| where ObjectName == "Memory"
| where CounterName == "% Committed Bytes In Use"
| summarize AvgMemory = avg(CounterValue) by Computer
| extend Status = case(AvgMemory > 80, "Critical", AvgMemory > 60, "Warning", "Healthy")
| project Computer, Metric="Memory", Value=round(AvgMemory,2), Status;

let DiskAlert =
Perf
| where TimeGenerated > ago(4h)
| where ObjectName == "LogicalDisk"
| where CounterName == "% Free Space"
| where InstanceName != "_Total"
| summarize AvgDiskFree = avg(CounterValue) by Computer, InstanceName
| extend Status = case(AvgDiskFree < 20, "Critical", AvgDiskFree < 40, "Warning", "Healthy")
| project Computer, Metric=strcat("Disk (", InstanceName, ")"), Value=round(AvgDiskFree,2), Status;

CPUAlert
| union MemoryAlert
| union DiskAlert
| order by Computer asc