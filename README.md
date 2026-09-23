WVDConnections
| where TimeGenerated >= ago(30m)
| where State == "Connected"
| where isnotempty(UserName)
| extend ComputerName = tolower(tostring(split(SessionHostName, ".")[0]))
| summarize UserName = strcat_array(make_set(UserName, 50), ", ") by ComputerName