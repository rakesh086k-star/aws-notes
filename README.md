Perf
| where TimeGenerated >= ago(1h)
| project
    TimeGenerated,
    Computer,
    ObjectName,
    CounterName,
    InstanceName,
    CounterValue
| take 50



Perf
| where TimeGenerated >= ago(1h)
| where ObjectName contains "Processor"
| where CounterName contains "Processor Time"
| project
    TimeGenerated,
    Computer,
    ObjectName,
    CounterName,
    InstanceName,
    CounterValue
| take 50


Perf
| where TimeGenerated >= ago(1h)
| where ObjectName contains "Processor"
| where CounterName contains "Processor Time"
| project
    TimeGenerated,
    Computer,
    ObjectName,
    CounterName,
    InstanceName,
    CounterValue
| take 50
