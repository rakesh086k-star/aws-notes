let SelectedUser = "Indresh340697@exlservice.com";
// सभी users के लिए:
// let SelectedUser = "";

let UserSessions =
WVDConnections
| where TimeGenerated >= ago(7d)
| where isnotempty(UserName)
| where State == "Connected"
| where isempty(SelectedUser) or UserName =~ SelectedUser
| extend
    TimeSlot = bin(TimeGenerated, 30m),
    HostKey = tolower(tostring(SessionHostName))
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
    HostKey = tolower(tostring(Computer))
| summarize
    CPU_Avg = round(avg(todouble(CounterValue)), 2),
    CPU_Max = round(max(todouble(CounterValue)), 2)
    by HostKey, TimeSlot;

let Memory =
Perf
| where TimeGenerated >= ago(7d)
| where ObjectName == "Memory"
| where CounterName == "Available MBytes"
| extend
    TimeSlot = bin(TimeGenerated, 30m),
    HostKey = tolower(tostring(Computer))
| summarize
    Available_Memory_Avg_MB =
        round(avg(todouble(CounterValue)), 2)
    by HostKey, TimeSlot;

UserSessions
| join kind=leftouter CPU
    on HostKey, TimeSlot
| join kind=leftouter Memory
    on HostKey, TimeSlot
| extend
    Health_Status =
        case(
            CPU_Max >= 90 or Available_Memory_Avg_MB < 2048,
            "Critical",
            CPU_Max >= 75 or Available_Memory_Avg_MB < 4096,
            "Warning",
            "Healthy"
        )
| project
    TimeSlot,
    UserName,
    CPU_Average = strcat(tostring(CPU_Avg), " %"),
    CPU_Maximum = strcat(tostring(CPU_Max), " %"),
    Available_Memory = strcat(
        tostring(round(Available_Memory_Avg_MB / 1024.0, 2)),
        " GB"
    ),
    Health_Status
| order by TimeSlot asc