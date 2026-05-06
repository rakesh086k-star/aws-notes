let CPUAlert =
Perf
| where TimeGenerated > ago(4h)
| where CounterName == "% Processor Time"
| summarize AvgCPU = avg(CounterValue) by Computer
| extend Status = case(
    AvgCPU > 80, "Critical",
    AvgCPU > 60, "Warning",
    "Healthy"
)
| project Computer, Metric="CPU", Value=round(AvgCPU,2), Status;