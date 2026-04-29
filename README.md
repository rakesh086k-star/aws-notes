Resources
| where type == "microsoft.compute/virtualmachines"
| extend osProfile = properties.osProfile
| project name, resourceGroup, location, osProfile