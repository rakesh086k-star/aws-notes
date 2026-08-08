LAWCustomfslogix_CL
| where TimeGenerated > ago(1h)
| project TimeGenerated, Computer, FilePath, RawData
| order by TimeGenerated desc
| take 20