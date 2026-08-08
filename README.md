LAWCustomFslogix_CL
| where TimeGenerated > ago(2h)
| extend RawText = tostring(RawData)
| where RawText has_any ("User:", "SizeInMBs", "MB left", "WindowsSessionID")
| project TimeGenerated, Computer, FilePath, RawText
| order by TimeGenerated desc