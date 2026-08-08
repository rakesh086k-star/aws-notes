LAWCustomfslogix_CL
| where TimeGenerated > ago(24h)
| summarize
    Total=count(),
    ComputerCount=dcount(Computer),
    FilePathCount=dcount(FilePath),
    RawDataCount=dcount(RawData)