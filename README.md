LAWCUSTOMFslogix_CL
| where isnotempty(tostring(RawData))
| project TimeGenerated, Computer, FilePath, RawData
| take 10