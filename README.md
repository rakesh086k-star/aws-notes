resources
| where type =~ "microsoft.authorization/locks"
| project name, properties.scope
