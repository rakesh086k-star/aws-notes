
let StartTime = datetime(2026-10-12 00:00:00);
let EndTime = datetime(2026-10-15 23:59:59);
let TargetUser = "Takshu255467@exlservice.com";
WVDErrors
| where TimeGenerated between (StartTime .. EndTime)
| where UserName =~ TargetUser
| project
    TimeGenerated,
    UserName,
    Code = Code,
    Message = Message
| order by TimeGenerated asc
