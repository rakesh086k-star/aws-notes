AALW_Custom_FSLogix_CLR_CL
| where TimeGenerated > ago(24h)
| project TimeGenerated, ComputerName, FilePath, RowData
| take 20