# Hi, I'm Rakesh 👋

## 🚀 About Me
- Cloud Engineer
- Learning AWS & DevOps
- Interested in Automation & Scripting

resourcecontainers
| where type == "microsoft.resources/subscriptions/resourcegroups"
| extend rgId = tolower(id)
| project resourceGroup = name, rgId
| join kind=leftouter (
    resources
    | where type == "microsoft.authorization/locks"
    | extend rgId = tolower(properties.scope)
    | project lockName = name, rgId, lockType = tostring(properties.level)
) on rgId
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
