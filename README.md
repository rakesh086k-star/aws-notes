let CPUAlert =
Perf
| where TimeGenerated > ago(4h)
| where CounterName == "% Processor Time"
| summarize AvgCPU = avg(CounterValue) by Computer
| where AvgCPU > 80
| project Computer, Metric="CPU", Value=round(AvgCPU,2);