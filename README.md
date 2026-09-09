let VMStatus =
arg("").Resources
| where type =~ "microsoft.compute/virtualmachines"
| where resourceGroup =~ "RG-AVD-PH-EI-US"
| extend
    ComputerName = tostring(name),
    PowerStateCode = tostring(properties.extended.instanceView.powerState.code)
| extend
    Status = case(
        PowerStateCode == "PowerState/running", "Online",
        PowerStateCode == "PowerState/stopped", "Stopped",
        PowerStateCode == "PowerState/deallocated", "Deallocated",
        "Unavailable"
    )
| project
    VMId = tolower(tostring(id)),
    ComputerName,
    Status;

let CurrentUsers =
WVDConnections
| where TimeGenerated >= ago(24h)
| summarize arg_max(TimeGenerated, *) by CorrelationId
| where State == "Connected"
| extend
    VMId = tolower(tostring(SessionHostAzureVmId)),
    ComputerName = tostring(split(SessionHostName, ".")[0]),
    UserName = tostring(UserName)
| project
    VMId,
    ComputerName,
    UserName;

VMStatus
| join kind=leftouter CurrentUsers on VMId
| project
    ComputerName,
    UserName = iff(isempty(UserName), "No User", UserName),
    Status
| order by case(
    Status == "Online", 1,
    Status == "Stopped", 2,
    Status == "Deallocated", 3,
    Status == "Unavailable", 4,
    5
) asc,
ComputerName asc





// VM Power State
let VMStatus =
arg("").Resources
| where type =~ "microsoft.compute/virtualmachines"
| where resourceGroup =~ "RG-AVD-PH-EI-US"
| extend
    ComputerName = tolower(tostring(name)),
    PowerStateCode = tostring(properties.extended.instanceView.powerState.code)
| extend
    Status = case(
        PowerStateCode == "PowerState/running", "Online",
        PowerStateCode == "PowerState/stopped", "Stopped",
        PowerStateCode == "PowerState/deallocated", "Deallocated",
        "Unavailable"
    )
| project ComputerName, Status;

// Current AVD Users
let CurrentUsers =
WVDConnections
| where TimeGenerated >= ago(24h)
| extend ComputerName = tolower(tostring(split(SessionHostName, ".")[0]))
| summarize arg_max(TimeGenerated, State) 
    by ComputerName, SessionHostSessionId, UserName
| where State == "Connected"
| project
    ComputerName,
    UserName;

// Combine VM Status + Current User
VMStatus
| join kind=leftouter CurrentUsers on ComputerName
| extend UserName = iff(isempty(UserName), "No User", UserName)
| project
    ComputerName,
    UserName,
    Status
| order by ComputerName asc










WVDConnections
| where TimeGenerated >= ago(6d)
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
| extend
    ComputerName = tostring(SessionHostName),
    UserName = tostring(UserName)
| summarize
    UniqueUsers = dcount(UserName)
    by LoginDate, DayName, ComputerName, UserName
| project
    DayName,
    LoginDate,
    ComputerName,
    UserName,
    UniqueUsers
| order by LoginDate asc, ComputerName asc, UserName asc

