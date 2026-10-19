
Heartbeat
| where TimeGenerated >= ago(24h)
| summarize arg_max(TimeGenerated, *) by Computer
| project Computer, OSType, OSName, OSMajorVersion,
          OSMinorVersion, LastSeen = TimeGenerated
| order by Computer asc
