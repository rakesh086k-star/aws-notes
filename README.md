Resources
| where type =~ "microsoft.desktopvirtualization/hostpools/sessionhosts"
| where id contains "/hostPools/AHP_PPH_US_EI_Pooled/"
| summarize TotalMachines = count()