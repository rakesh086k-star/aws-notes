Heartbeat
| where TimeGenerated >= ago(24h)
| summarize arg_max(TimeGenerated, *) by Computer
| extend OSVersion = tostring(column_ifexists("OSVersion", "")),
         OSName = tostring(column_ifexists("OSName", ""))
| project Computer, OSName, OSVersion,
          OSType, LastSeen = TimeGenerated
| order by Computer asc