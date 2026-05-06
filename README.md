let CPUAlert = 
Perf
| where ObjectName == "Processor"
| where CounterName == "% Processor Time"
| where TimeGenerated {TimeRange}
| summarize AvgCPU = avg(CounterValue) by Computer
| where AvgCPU > 80;

let MemAlert =
Perf
| where ObjectName == "Memory"
| where CounterName == "% Committed Bytes In Use"
| where TimeGenerated {TimeRange}
| summarize AvgMem = avg(CounterValue) by Computer
| where AvgMem > 80;

CPUAlert
| project Type="CPU", Computer, Value=AvgCPU
| union (MemAlert | project Type="Memory", Computer, Value=AvgMem)