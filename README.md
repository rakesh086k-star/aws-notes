Perf
| where TimeGenerated > ago(4h)
| where CounterName == "% Processor Time"
| take 20