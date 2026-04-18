# Hi, I'm Rakesh 👋

## 🚀 About Me
- , OperationNameValue, ActivityStatusValue

let locks = resources
| where type == "microsoft.authorization/locks"
| extend resourceGroup = tostring(split(id, "/")[4])
| project resourceGroup, lockType = tostring(properties.level), lockName = name;

resourcecontainers
| where type == "microsoft.resources/subscriptions/resourcegroups"
| project resourceGroup = name
| join kind=leftouter locks on resourceGroup
| summarize lockType = any(lockType) by resourceGroup
| extend lockStatus = iff(isempty(lockType), "No Lock", lockType)
| order by resourceGroup asc

## 🛠 Skills
- AWS
- Linux
- PowerShell / Bash
- Networking Basics

## 📂 Projects
- Coming Soon...

## 📫 Contact
- Email: rakesh086.k@gmail.com
