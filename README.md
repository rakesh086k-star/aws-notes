let SelectedUser = "";
// All users = blank
// Specific user example:
// let SelectedUser = "Anuradha218532@exlservice.com";

let UserSessions =
WVDConnections
| where TimeGenerated >= ago(7d)
| where State == "Connected"
| where isnotempty(UserName)
| distinct CorrelationId, UserName, SessionHostName
| where isempty(SelectedUser) or UserName =~ SelectedUser
| extend HostName = tolower(tostring(split(SessionHostName, ".")[0]));

let HostMetrics =
AzureMetrics
| where TimeGenerated >= ago(7d)
| where MetricName in (
    "Percentage CPU",
    "Available Memory Percentage",
    "Network In Total",
    "Network Out Total"
)
| extend
    TimeSlot = bin(TimeGenerated, 30m),
    HostName = tolower(Resource)
| summarize
    Avg_CPU = avgif(Average, MetricName == "Percentage CPU"),
    Max_CPU = maxif(Maximum, MetricName == "Percentage CPU"),

    Avg_Available_Memory =
        avgif(Average, MetricName == "Available Memory Percentage"),

    Min_Available_Memory =
        minif(Minimum, MetricName == "Available Memory Percentage"),

    Network_In_Bytes =
        sumif(Total, MetricName == "Network In Total"),

    Network_Out_Bytes =
        sumif(Total, MetricName == "Network Out Total")
    by HostName, TimeSlot;

UserSessions
| join kind=inner HostMetrics
    on HostName
| extend
    CPU_Status =
        case(
            Avg_CPU >= 90, "Critical",
            Avg_CPU >= 75, "Warning",
            "Healthy"
        ),

    Memory_Status =
        case(
            Avg_Available_Memory <= 10, "Critical",
            Avg_Available_Memory <= 20, "Warning",
            "Healthy"
        ),

    Overall_Status =
        case(
            Avg_CPU >= 90 or Avg_Available_Memory <= 10,
            "Critical",

            Avg_CPU >= 75 or Avg_Available_Memory <= 20,
            "Warning",

            "Healthy"
        )
| extend
    CPU_Display =
        strcat(tostring(round(Avg_CPU, 2)), " %"),

    Max_CPU_Display =
        strcat(tostring(round(Max_CPU, 2)), " %"),

    Available_Memory_Display =
        strcat(tostring(round(Avg_Available_Memory, 2)), " %"),

    Min_Available_Memory_Display =
        strcat(tostring(round(Min_Available_Memory, 2)), " %"),

    Network_In_Display =
        strcat(
            tostring(round(Network_In_Bytes / 1024.0 / 1024.0, 2)),
            " MB"
        ),

    Network_Out_Display =
        strcat(
            tostring(round(Network_Out_Bytes / 1024.0 / 1024.0, 2)),
            " MB"
        )
| project
    TimeSlot,
    UserName,
    CPU = CPU_Display,
    Max_CPU = Max_CPU_Display,
    Available_Memory = Available_Memory_Display,
    Min_Available_Memory = Min_Available_Memory_Display,
    Network_In = Network_In_Display,
    Network_Out = Network_Out_Display,
    CPU_Status,
    Memory_Status,
    Overall_Status
| order by TimeSlot asc