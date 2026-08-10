LAWCUSTOMFslogix_CL
| summarize
    TotalRecords = count(),
    RawDataPresent = countif(isnotempty(tostring(RawData))),
    RawDataEmpty = countif(isempty(tostring(RawData)))