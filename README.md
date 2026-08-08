FSLogixProfile_CL
| where TimeGenerated > ago(30m)
| project TimeGenerated, Computer, FilePath, RawData
| order by TimeGenerated desc