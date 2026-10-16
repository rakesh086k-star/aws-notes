
let StartTime = datetime(2026-09-15 00:00:00);
let EndTime = datetime(2026-10-16 23:59:59);
let TargetUser = "Takshu255467@exlservice.com";
let Errors =
    WVDErrors
    | where TimeGenerated between (StartTime .. EndTime)
    | where UserName =~ TargetUser
    | project
        TimeGenerated,
        UserName,
        CodeSymbolic,
        Message;
Errors
| extend TotalErrorEvents = toscalar(Errors | summarize count())
| order by TimeGenerated desc
