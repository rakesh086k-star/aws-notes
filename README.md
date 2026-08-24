desktopvirtualizationresources
| where type =~ "microsoft.desktopvirtualization/hostpools/sessionhosts"
| extend HostPoolName = tostring(split(name, "/")[0])
| where HostPoolName =~ "AHP_PPH_US_EI_Pooled"
| summarize TotalMachines = count()