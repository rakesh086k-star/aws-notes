let SelectedUser = "Indresh340697@exlservice.com";
// All users के लिए: let SelectedUser = "";

let UserSessions =
WVDConnections
| where TimeGenerated >= ago(7d)
| where isnotempty(UserName)
| where isempty(SelectedUser) or UserName =~ SelectedUser
| extend
    TimeSlot = bin(TimeGenerated, 30m),
    HostKey = tolower(tostring(split(SessionHostName, ".")[0]))
| summarize by UserName, HostKey, TimeSlot;

let CPU =
Perf
| where TimeGenerated >= ago(7d)
| where ObjectName == "Processor Information"
| where CounterName == "% Processor Time"
| where InstanceName == "_Total"
| extend
    TimeSlot = bin(TimeGenerated, 30m),
    HostKey = tolower(tostring(split(Computer, ".")[0]))
| summarize
    CPU_Avg = round(avg(todouble(CounterValue)), 2),
    CPU_Max = round(max(todouble(CounterValue)), 2)
    by HostKey, TimeSlot;

let Memory =
InsightsMetrics
| where TimeGenerated >= ago(7d)
| where Origin == "vm.azm.ms"
| where Namespace == "Memory"
| where Name == "AvailableMB"
| extend
    TotalMemoryMB = toreal(todynamic(Tags)["vm.azm.ms/memorySizeMB"]),
    AvailableMB = toreal(Val),
    TimeSlot = bin(TimeGenerated, 30m),
    HostKey = tolower(tostring(split(Computer, ".")[0]))
| where isnotnull(TotalMemoryMB) and TotalMemoryMB > 0
| extend AvailableMemoryPercent =
    (AvailableMB / TotalMemoryMB) * 100.0
| summarize
    Available_Memory_Avg_Percent =
        round(avg(AvailableMemoryPercent), 2)
    by HostKey, TimeSlot;

UserSessions
| join kind=leftouter CPU
    on HostKey, TimeSlot
| join kind=leftouter Memory
    on HostKey, TimeSlot
| extend Health_Status =
    case(
        CPU_Max >= 90 or Available_Memory_Avg_Percent < 10,
        "Critical",
        CPU_Max >= 75 or Available_Memory_Avg_Percent < 20,
        "Warning",
        "Healthy"
    )
| project
    TimeSlot,
    Computer = HostKey,
    UserName,
    CPU_Average = strcat(tostring(CPU_Avg), " %"),
    CPU_Maximum = strcat(tostring(CPU_Max), " %"),
    Available_Memory_Average =
        strcat(tostring(Available_Memory_Avg_Percent), " %"),
    Health_Status
| order by TimeSlot asc