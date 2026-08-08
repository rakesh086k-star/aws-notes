FSLogixProfile_CL
| where TimeGenerated > ago(24h)
| project TimeGenerated, Computer, RawData
| order by TimeGenerated desc