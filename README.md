ASRReplicationItems
| take 10


ASRReplicationItems
| order by TimeGenerated desc


search *
| summarize Count=count() by $table
| order by Count desc


search *
| where $table contains "Recovery"
   or $table contains "SiteRecovery"
   or $table contains "AzureActivity"
| summarize Count=count() by $table
| order by Count desc