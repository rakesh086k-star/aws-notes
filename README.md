LAWCustomFslogix_CL
| where TimeGenerated > ago(24h)
| where strlen(RawData) > 10
| project TimeGenerated, Computer, RawData
| order by TimeGenerated desc
| take 100