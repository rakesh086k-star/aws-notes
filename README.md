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
| summarize
    by UserName, HostKey, TimeSlot;

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

let AvailableMemory =
Perf
| where TimeGenerated >= ago(7d)
| where ObjectName == "Memory"
| where CounterName in ("Available MBytes", "Total Visible Memory Size")
| extend
    TimeSlot = bin(TimeGenerated, 30m),
    HostKey = tolower(tostring(split(Computer, ".")[0]))
| summarize
    MemoryValue = avg(todouble(CounterValue))
    by HostKey, TimeSlot, CounterName
| summarize
    AvailableMB = maxif(MemoryValue, CounterName == "Available MBytes"),
    TotalMemoryMB = maxif(MemoryValue, CounterName == "Total Visible Memory Size")
    by HostKey, TimeSlot
| extend
    Available_Memory_Avg_Percent =
        round((AvailableMB / TotalMemoryMB) * 100, 2);

UserSessions
| join kind=leftouter CPU
    on HostKey, TimeSlot
| join kind=leftouter AvailableMemory
    on HostKey, TimeSlot
| extend
    Health_Status =
        case(
            CPU_Max >= 90 or Available_Memory_Avg_Percent < 10, "Critical",
            CPU_Max >= 75 or Available_Memory_Avg_Percent < 20, "Warning",
            "Healthy"
        )
| project
    TimeSlot,
    Computer = HostKey,
    UserName,
    CPU_Average_Percent = strcat(tostring(CPU_Avg), " %"),
    CPU_Maximum_Percent = strcat(tostring(CPU_Max), " %"),
    Available_Memory_Average_Percent =
        strcat(tostring(Available_Memory_Avg_Percent), " %"),
    Health_Status
| order by TimeSlot asc