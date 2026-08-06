search *
| where $table contains "Recovery"
or $table contains "Site"
or $table contains "Replication"
| summarize count() by $table