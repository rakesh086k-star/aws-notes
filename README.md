Resources
| where type =~ "microsoft.desktopvirtualization/hostpools/sessionhosts"
| project name, id, resourceGroup
| limit 20