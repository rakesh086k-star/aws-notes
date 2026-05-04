let vmList = Heartbeat
| where ResourceGroup == "<YOUR-RG-NAME>"
| summarize arg_max(TimeGenerated, *) by Computer;

vmList
| extend Status = case(
    OSType == "Windows" and isnotempty(Computer), "Running",
    isempty(Computer), "Unavailable",
    "Stopped"
)
| summarize 
    TotalSystems = count(),
    Running = countif(Status == "Running"),
    Stopped = countif(Status == "Stopped"),
    Unavailable = countif(Status == "Unavailable")


    Resources
| where type == "microsoft.compute/virtualmachines"
| where resourceGroup == "<YOUR-RG-NAME>"
| extend powerState = tostring(properties.extended.instanceView.powerState.code)
| summarize 
    Total = count(),
    Running = countif(powerState == "PowerState/running"),
    Stopped = countif(powerState contains "stopped"),
    Deallocated = countif(powerState contains "deallocated")


    QUERY 2: Users (Active / Idle / Disconnected)

let connected =
WVDConnections
| where State == "Connected"
| extend IdleMinutes = datetime_diff("minute", now(), TimeGenerated)
| extend Status = iff(IdleMinutes > 10, "Idle", "Active")
| summarize Count = dcount(UserName) by Status;

let disconnected =
WVDConnections
| where State == "Disconnected"
| summarize Count = dcount(UserName)
| extend Status = "Disconnected";

connected
| union disconnected


QUERY 3: System Health (Healthy / Warning / Critical)

let cpu = Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| summarize avgCPU = avg(CounterValue) by Computer;

cpu
| extend Health = case(
    avgCPU < 60, "Healthy",
    avgCPU < 80, "Warning",
    "Critical"
)
| summarize Count = count() by Health


AzureDiagnostics
| where Level in ("Error","Critical","Warning")
| project TimeGenerated, Resource, Level, OperationName, ResultDescription
| sort by TimeGenerated desc
| take 10


--------------------

Resources
| where type == "microsoft.compute/virtualmachines"
| where resourceGroup =~ "<YOUR-RG-NAME>"
| extend powerState = tostring(properties.extended.instanceView.powerState.code)
| extend Status = case(
    powerState == "PowerState/running", "Running",
    powerState == "PowerState/stopped", "Stopped",
    powerState == "PowerState/deallocated", "Deallocated",
    "Unavailable"
)
| summarize 
    Total = count(),
    Running = countif(Status == "Running"),
    Stopped = countif(Status == "Stopped"),
    Deallocated = countif(Status == "Deallocated"),
    Unavailable = countif(Status == "Unavailable")
