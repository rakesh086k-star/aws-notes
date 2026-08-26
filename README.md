Resources
| where type =~ "microsoft.desktopvirtualization/hostpools"
| project name, resourceGroup, subscriptionId, location, id