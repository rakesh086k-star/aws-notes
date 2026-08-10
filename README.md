AALW_Custom_FSLogix_CL
| summarize
    TotalRecords = count(),
    Critical = countif(Severity_s =~ "Critical"),
    Warning = countif(Severity_s =~ "Warning"),
    Errors = countif(Severity_s =~ "Error")