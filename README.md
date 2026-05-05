WVDConnections
| where TimeGenerated > ago(1d)
| summarize DailyUsers = dcount(UserName)
| extend WeeklyProjection = DailyUsers * 5
| extend MonthlyProjection = DailyUsers * 22
| extend DailyActual = DailyUsers
| project DailyActual, WeeklyProjection, 

WVDConnections
| where TimeGenerated > ago(7d)
| extend LoginDate = startofday(TimeGenerated)
| extend DayName = format_datetime(TimeGenerated, "dddd")
| summarize UniqueUsers = dcount(UserName) by LoginDate, DayName
| order by LoginDate 
WVDConnections
| where TimeGenerated > ago(7d)
| extend LoginDate = startofday(TimeGenerated)
| extend DayName = format_datetime(LoginDate, "dddd")
| summarize UniqueUsers = dcount(UserName) by LoginDate, DayName
| order by LoginDate 


WVDConnections
| where TimeGenerated > ago(7d)
| extend LoginDate = startofday(TimeGenerated)
| extend DayNum = dayofweek(LoginDate)
| extend DayName = case(
    DayNum == 0d, "Sunday",
    DayNum == 1d, "Monday",
    DayNum == 2d, "Tuesday",
    DayNum == 3d, "Wednesday",
    DayNum == 4d, "Thursday",
    DayNum == 5d, "Friday",
    DayNum == 6d, "Saturday",
    "Unknown"
)
| summarize UniqueUsers = dcount(UserName) by LoginDate, DayName
| order by LoginDate 

WVDConnections
| where TimeGenerated > ago(7d)
| extend LoginDate = startofday(TimeGenerated)
| extend DayName = case(
    dayofweek(LoginDate) == 0d, "Sunday",
    dayofweek(LoginDate) == 1d, "Monday",
    dayofweek(LoginDate) == 2d, "Tuesday",
    dayofweek(LoginDate) == 3d, "Wednesday",
    dayofweek(LoginDate) == 4d, "Thursday",
    dayofweek(LoginDate) == 5d, "Friday",
    dayofweek(LoginDate) == 6d, "Saturday",
    "Unknown"
)
| summarize UniqueUsers = dcount(UserName) by LoginDate, DayName
| project LoginDate, DayName, UniqueUsers
| order by LoginDate asc


let cpuThreshold = 80;
let diskThreshold = 15;

union isfuzzy=true

// ===================== CPU HIGH =====================
(
    Perf
    | where ObjectName == "Processor"
    | where CounterName == "% Processor Time"
    | summarize Value = avg(CounterValue) by TimeGenerated, Computer, _ResourceId
    | where Value > cpuThreshold
    | extend MetricType = "CPU"
    | extend Severity = "🔴 CRITICAL"
    | extend Issue = "High CPU Utilization"
),

// ===================== DISK LOW =====================
(
    Perf
    | where ObjectName == "LogicalDisk"
    | where CounterName == "% Free Space"
    | summarize Value = avg(CounterValue) by TimeGenerated, Computer, InstanceName, _ResourceId
    | where Value < diskThreshold
    | extend MetricType = "Disk"
    | extend Severity = "🟠 WARNING"
    | extend Issue = "Low Disk Space"
),

// ===================== SESSION ISSUES =====================
(
    WVDCheckpoints
    | where Message has "Disconnected" or State == "Disconnected"
    | summarize Value = count() by TimeGenerated, SessionHostName, HostPoolName, _ResourceId
    | extend Computer = SessionHostName
    | extend MetricType = "Session"
    | extend Severity = "🔵 INFO"
    | extend Issue = "User Session Disconnected"
)

// ===================== FINAL STANDARD FORMAT =====================
| project
    TimeGenerated,
    Computer,
    MetricType,
    Value,
    Severity,
    Issue
| order by TimeGenerated desc

Event
| where Source == "FSLogix Apps"
| extend MessageText = tostring(RenderedDescription)

// ---------------- Extract VM ----------------
| extend VM = Computer

// ---------------- Extract User (best effort) ----------------
| extend User = extract(@"User:\s*([a-zA-Z0-9._-]+)", 1, MessageText)
| extend User = iff(isempty(User), "Unknown", User)

// ---------------- Profile Size (if present in logs) ----------------
| extend ProfileSizeGB = todouble(extract(@"(\d+(\.\d+)?)\s*GB", 1, MessageText))
| extend ProfileSize = iff(isnull(ProfileSizeGB), "Not Available", strcat(tostring(ProfileSizeGB), " GB"))

// ---------------- Action detection ----------------
| extend Action = case(
    MessageText has "Mounted", "Mounted",
    MessageText has "Attach", "Attach",
    MessageText has "Unmounted", "Unmounted",
    MessageText has "Detach", "Detach",
    "Unknown"
)

// ---------------- Status logic ----------------
| extend Status = case(
    MessageText has "Failed" or MessageText has "Error", "Critical",
    MessageText has "slow" or MessageText has "Slow", "Warning",
    Action == "Mounted", "Healthy",
    "Info"
)

// ---------------- FINAL OUTPUT ----------------
| project
    TimeGenerated,
    VM,
    User,
    ProfileSize,
    Status,
    Action
| order by TimeGenerated desc

let cpuThreshold = 80;
let diskThreshold = 15;

union isfuzzy=true

// ===================== CPU HIGH =====================
(
    Perf
    | where ObjectName == "Processor"
    | where CounterName == "% Processor Time"
    | summarize Value = avg(CounterValue) by bin(TimeGenerated, 5m), Computer, _ResourceId
    | where Value > cpuThreshold
    | extend MetricType = "CPU"
    | extend Severity = "🔴 CRITICAL"
    | extend Issue = "High CPU Utilization"
),

// ===================== DISK LOW =====================
(
    Perf
    | where ObjectName == "LogicalDisk"
    | where CounterName == "% Free Space"
    | summarize Value = avg(CounterValue) by bin(TimeGenerated, 5m), Computer, InstanceName, _ResourceId
    | where Value < diskThreshold
    | extend MetricType = "Disk"
    | extend Severity = "🟠 WARNING"
    | extend Issue = "Low Disk Space"
    | extend Computer = strcat(Computer, " (", InstanceName, ")")
),

// ===================== SESSION ISSUES =====================
(
    WVDCheckpoints
    | where Message has "Disconnected" or State == "Disconnected"
    | summarize Value = count() by bin(TimeGenerated, 5m), SessionHostName, HostPoolName, _ResourceId
    | extend Computer = SessionHostName
    | extend MetricType = "Session"
    | extend Severity = "🔵 INFO"
    | extend Issue = "User Session Disconnected"
)

// ===================== FINAL FORMAT =====================
| project
    TimeGenerated,
    Computer,
    MetricType,
    Value,
    Severity,
    Issue
| order by TimeGenerated desc



