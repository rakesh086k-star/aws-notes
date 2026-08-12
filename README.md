Perf
| where TimeGenerated >= ago(1h)
| where ObjectName contains "Processor"
| project
    TimeGenerated,
    Computer,
    ObjectName,
    CounterName,
    InstanceName,
    CounterValue
| take 100