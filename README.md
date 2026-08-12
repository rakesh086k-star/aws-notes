Perf
| where TimeGenerated >= ago(1d)
| where ObjectName contains "Processor"
| project TimeGenerated, Computer, ObjectName, CounterName, InstanceName, CounterValue
| take 50