let SelectedUser = "";
// Specific user example:
// let SelectedUser = "Indresh340697@exlservice.com";
// Blank = all users

let UserSessions =
WVDConnections
| where TimeGenerated >= ago(7d)
| where isnotempty(UserName)
| where State == "Connected"
| where isempty(SelectedUser) or UserName =~ SelectedUser
| extend
    TimeSlot = bin(TimeGenerated, 30m),
    HostKey = tolower(tostring(split(SessionHostName, ".")[0]))
| summarize
    by UserName, HostKey, TimeSlot;

let CPU =
Perf
| where TimeGenerated >= ago(7d)
| where ObjectName == "Processor"
| where CounterName == "% Processor Time"
| where InstanceName == "_Total"
| extend
    TimeSlot = bin(TimeGenerated, 30m),
    HostKey = tolower(tostring(split(Computer, ".")[0]))
| summarize
    CPU_Avg = round(avg(CounterValue), 2),
    CPU_Max = round(max(CounterValue), 2)
    by HostKey, TimeSlot;

let Memory =
Perf
| where TimeGenerated >= ago(7d)
| where ObjectName == "Memory"
| where CounterName == "Available MBytes"
| extend
    TimeSlot = bin(TimeGenerated, 30m),
    HostKey = tolower(tostring(split(Computer, ".")[0]))
| summarize
    Available_Memory_Avg_MB = round(avg(CounterValue), 2),
    Available_Memory_Min_MB = round(min(CounterValue), 2)
    by HostKey, TimeSlot;

UserSessions
| join kind=leftouter CPU
    on HostKey, TimeSlot
| join kind=leftouter Memory
    on HostKey, TimeSlot
| extend
    CPU_Avg_Display =
        iff(isnull(CPU_Avg), "N/A",
            strcat(tostring(CPU_Avg), " %")),

    CPU_Max_Display =
        iff(isnull(CPU_Max), "N/A",
            strcat(tostring(CPU_Max), " %")),

    Available_Memory_Avg_Display =
        iff(isnull(Available_Memory_Avg_MB), "N/A",
            strcat(tostring(Available_Memory_Avg_MB), " MB")),

    Available_Memory_Min_Display =
        iff(isnull(Available_Memory_Min_MB), "N/A",
            strcat(tostring(Available_Memory_Min_MB), " MB"))
| project
    TimeSlot,
    UserName,
    CPU_Avg = CPU_Avg_Display,
    CPU_Max = CPU_Max_Display,
    Available_Memory_Avg = Available_Memory_Avg_Display,
    Available_Memory_Min = Available_Memory_Min_Display
| order by TimeSlot asc