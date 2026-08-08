LAWCustomFslogix_CL
| where TimeGenerated > ago(24h)
| extend RawText = tostring(RawData)
| where RawText has "LoadProfile"
| project TimeGenerated, Computer, RawText
| order by TimeGenerated desc