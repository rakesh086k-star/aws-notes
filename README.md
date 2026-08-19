Perf
| where TimeGenerated > ago(12h)
| summarize LastSeen = max(TimeGenerated) by Computer
| extend MinutesAgo = datetime_diff('minute', now(), LastSeen)
| summarize
    TotalMachines = count(),
    ActiveLast15Min = countif(MinutesAgo <= 15),
    ActiveLast30Min = countif(MinutesAgo <= 30),
    ActiveLast1Hour = countif(MinutesAgo <= 60),
    ActiveLast2Hours = countif(MinutesAgo <= 120)