let CurrentUsers =
WVDConnections
| where TimeGenerated >= ago(30m)
| where State == "Connected"
| where isnotempty(UserName)
| extend ComputerName = tolower(tostring(split(SessionHostName, ".")[0]))
| summarize UserName = strcat_array(make_set(UserName, 50), ", ")
    by ComputerName;

let ResourceUtilization =
Perf
| where TimeGenerated >= ago(15m)
| extend ComputerName = tolower(tostring(split(Computer, ".")[0]))
| summarize
    CPU_Utilization = round(avgif(
        CounterValue,
        ObjectName == "Processor Information"
        and CounterName == "% Processor Time"
        and InstanceName == "_Total"
    ), 2),

    Memory_Utilization = round(avgif(
        CounterValue,
        ObjectName == "Memory"
        and CounterName == "% Committed Bytes In Use"
    ), 2),

    Disk_Free = round(avgif(
        CounterValue,
        ObjectName == "LogicalDisk"
        and CounterName == "% Free Space"
        and InstanceName == "_Total"
    ), 2)
    by ComputerName;

ResourceUtilization
| extend Disk_Utilization = round(100.0 - Disk_Free, 2)
| join kind=leftouter CurrentUsers on ComputerName
| extend UserName = iff(isempty(UserName), "No User", UserName)
| project
    ComputerName,
    UserName,
    CPU_Utilization = strcat(tostring(CPU_Utilization), "%"),
    Memory_Utilization = strcat(tostring(Memory_Utilization), "%"),
    Disk_Utilization = strcat(tostring(Disk_Utilization), "%")
| order by ComputerName asc