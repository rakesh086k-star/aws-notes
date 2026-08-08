LAWCUSTOMfslogix_CL
| where TimeGenerated > ago(24h)
| project
    TimeGenerated,
    Computer,
    FilePath,
    RawData,
    RawDataType = gettype(RawData)
| take 10