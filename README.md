# Hi, I'm Rakesh 👋

## 🚀 About Me
- Cloud Engineer
- Learning AWS & DevOps
- Interested in Automation & Scripting

resourcecontainers
| where type == "microsoft.resources/subscriptions/resourcegroups"
| project resourceGroup=name, subscriptionId
| join kind=leftouter (
    resources
    | where type == "microsoft.authorization/locks"
    | project lockName=name, resourceGroup, lockType=tostring(properties.level)
) on resourceGroup
| extend lockStatus = iff(isempty(lockName), "No Lock", lockType)
| project resourceGroup, lockStatus
| order by resourceGroup asc

AzureActivity
| where OperationNameValue contains "Microsoft.Authorization/locks"
| project TimeGenerated, ResourceGroup, OperationNameValue, ActivityStatusValue


## 🛠 Skills
- AWS
- Linux
- PowerShell / Bash
- Networking Basics

## 📂 Projects
- Coming Soon...

## 📫 Contact
- Email: rakesh086.k@gmail.com
