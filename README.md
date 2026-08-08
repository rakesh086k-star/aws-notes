AALCUSTOMFslogix_CL
| where TimeGenerated > ago(24h)
| extend RawText = tostring(RawData)
| project TimeGenerated, Computer, FilePath, RawText
| order by TimeGenerated desc