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




let cpuThreshold = 80;
let diskThreshold = 15;

// ===================== CPU =====================
let cpu =
Perf
| where TimeGenerated > ago(24h)
| where ObjectName == "Processor"
| where CounterName == "% Processor Time"
| summarize Value = avg(CounterValue) by bin(TimeGenerated, 5m), Computer, _ResourceId
| where isnotempty(Value)
| where Value > cpuThreshold
| extend MetricType = "CPU"
| extend Severity = "🔴 CRITICAL"
| extend Issue = "High CPU Utilization";

// ===================== DISK =====================
let disk =
Perf
| 
Hi [Engineer Name],

AVD admin access has now been enabled for you. You are expected to take ownership of BAU activities and manage day-to-day operations without any failures or escalations.

This email is to formally track your progress in this role.

You have completed around 2 years in the organization, however based on my observations, there is still a gap in handling assigned operational tasks and overall understanding of AVD infrastructure. You need to focus on improving your technical skills and strengthen your knowledge to meet the role expectations.

Please work on these areas and show improvement in the coming weeks.

Bharat is added for visibility.

Thanks,  
[Your Name]



hope you are doing well. I wanted to share some feedback from the recent KT sessions. I’ve noticed that some topics need to be repeated multiple times, which is slowing down your progress. At the moment, your progress is slower than expected and not at the level we are aiming for.

Also, during one of the sessions, when I was explaining and my tone became slightly louder, you started to cry. I understand it may have felt uncomfortable, but my intention was only to help you understand the topic better.

Additionally, I feel there is a lack of focused study on AVD concepts. It’s important to keep your main focus on understanding these concepts so that your basics remain clear and strong.

It’s important to stay calm and try to understand step by step, even if something feels difficult. Please feel free to ask questions, but also try to practice and revisit the concepts on your own. This will help you improve faster and build more confidence in your work.





