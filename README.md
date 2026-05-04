Perf
| where ObjectName == "Processor"
| where CounterName == "% Processor Time"
| where InstanceName == "_Total"
| summarize CPU_Usage = avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
| sort by TimeGenerated desc


Perf
| where ObjectName == "Memory"
| where CounterName == "% Committed Bytes In Use"
| summarize Memory_Usage = avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
| sort by TimeGenerated desc

WVDConnections
| where State == "Connected"
| extend LoginTime = datetime_diff('second', TimeGenerated, PredecessorConnectionTime)
| summarize AvgLoginTime = avg(LoginTime) by Computer, bin(TimeGenerated, 5m)
| sort by AvgLoginTime desc

Perf
| where ObjectName == "LogicalDisk"
| where CounterName == "Free Megabytes"
| extend FreeGB = CounterValue / 1024
| summarize arg_max(TimeGenerated, *) by Computer, InstanceName
| project Computer, Drive=InstanceName, FreeGB

Event
| where Source == "FSLogix"
| where EventLevelName in ("Error","Warning")
| summarize Count = count() by Computer, EventLevelName, bin(TimeGenerated, 10m)

Perf
| where CounterName == "Free Megabytes"
| extend FreeGB = CounterValue / 1024
| extend Status = case(
    FreeGB < 5, "🔴 Critical",
    FreeGB < 10, "🟠 Warning",
    FreeGB < 20, "🟡 Low",
    "🟢 Healthy"
)
| summarize arg_max(TimeGenerated, *) by Computer
| project Computer, FreeGB, Status

WVDConnections
| summarize Sessions = count() by Computer
| join kind=leftouter (
    Event
    | where Source == "FSLogix"
    | summarize Errors = count() by Computer
) on Computer


WVDConnections
| summarize UserCount=dcount(UserName) by UserName, GatewayRegion, SourceSystem
| extend GatewayRegion = case(
    GatewayRegion == "eastus", "🔵 East US",
    GatewayRegion == "eastasia", "🟢 East Asia",
    GatewayRegion == "eastus2", "🟠 East US 2",
    GatewayRegion == "centralindia", "🟣 Central India",
    GatewayRegion
)



--------Dashboard---------

Heartbeat
| summarize TotalSystems = dcount(Computer)

-----------------------------------

let cpu = Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| summarize avgCPU = avg(CounterValue) by Computer;

cpu
| extend Status = case(
    avgCPU < 60, "Healthy",
    avgCPU < 80, "Warning",
    "Critical"
)
| summarize Count = count() by Status

---------------------------------------

Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| summarize avgCPU = avg(CounterValue)


______________________________

Perf
| where ObjectName == "Memory" and CounterName == "% Committed Bytes In Use"
| summarize avgMemory = avg(CounterValue)

-------

Perf
| where ObjectName == "LogicalDisk" and CounterName == "% Free Space"
| summarize avgFree = avg(CounterValue)
| extend DiskUsed = 100 - avgFree

-----
WVDConnections
| summarize ActiveSessions = count()
--------

Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| summarize avgCPU = avg(CounterValue)

--------
Connection Errors

WVDConnections
| where State == "Failed"
| summarize Errors = count() by bin(TimeGenerated, 5m)

-----
recent alreat 
AzureDiagnostics
| where Level == "Error"
| project TimeGenerated, Resource, OperationName, ResultDescription
| sort by TimeGenerated desc
  
------
10. Top Systems by Resource Usage
Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| summarize avgCPU = avg(CounterValue) by Computer
| top 5 by avgCPU desc

---------------------

| where State == "Connected"
| extend IdleTime = datetime_diff("minute", now(), TimeGenerated)
| extend Status = iff(IdleTime > 10, "Idle", "Active")
| summarize Users = dcount(UserName) by Status;

let disconnected =
WVDConnections
| where State == "Disconnected"
| summarize Users = dcount(UserName)
| extend Status = "Disconnected";

connected
| union disconnected
