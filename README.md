let StartTime = datetime(2026-09-17 00:00:00);
let EndTime = datetime(2026-09-17 23:59:59);
let TargetUser = "sritchie_wo@exlservice.com";

WVDErrors
| where TimeGenerated between (StartTime .. EndTime)
| where UserName =~ TargetUser
| project TimeGenerated, UserName, CodeSymbolic, Message, CorrelationId
| order by TimeGenerated asc