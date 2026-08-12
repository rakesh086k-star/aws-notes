LAWCustomFslogix_CL
| where TimeGenerated > ago(24h)
| project TimeGenerated, Computer, FilePath, RawData
| extend RawDataLength = strlen(RawData)
| extend RawDataPreview = substring(RawData, 0, 500)
| take 20