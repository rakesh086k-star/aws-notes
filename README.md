LAWCustomfslogix_CL
| where TimeGenerated > ago(24h)
| take 1
| project TimeGenerated, Computer, FilePath, RawData