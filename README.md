ASRReplicatedItems
| summarize Machines = dcount(ReplicatedItemUniqueId)
    by PrimaryFabricName, RecoveryRegion
| order by Machines desc




ASRReplicatedItems
| summarize arg_max(TimeGenerated, *) by ReplicatedItemUniqueId
| extend HealthStatus = case(
    isnotempty(ReplicationHealthErrors) and 
        (ReplicationStatus =~ "Failed" or ReplicationStatus =~ "Error"),
        "Critical",

    isnotempty(ReplicationHealthErrors),
        "Warning",

    ReplicationStatus =~ "Failed" or ReplicationStatus =~ "Error",
        "Critical",

    ReplicationStatus =~ "Protected" or ReplicationStatus =~ "Healthy",
        "Healthy",

    "Warning"
)
| summarize Machines = count() by HealthStatus
| order by 
    case(
        HealthStatus == "Critical", 1,
        HealthStatus == "Warning", 2,
        HealthStatus == "Healthy", 3,
        4
    ) asc






ASRReplicatedItems
| summarize arg_max(TimeGenerated, *) by ReplicatedItemUniqueId
| extend HealthStatus = case(
    ReplicationStatus =~ "Failed" or ReplicationStatus =~ "Error", "Critical",
    isnotempty(ReplicationHealthErrors), "Error",
    ReplicationStatus =~ "Protected" or ReplicationStatus =~ "Healthy", "Healthy",
    "Warning"
)
| summarize Machines = count() by HealthStatus
| order by Machines desc



ASRReplicatedItems
| summarize arg_max(TimeGenerated, *) by ReplicatedItemUniqueId
| summarize Machines = count() by FailoverReadiness
| order by Machines desc



ASRReplicatedItems
| summarize arg_max(TimeGenerated, *) by ReplicatedItemUniqueId
| extend TestStatus = iff(
    isnotempty(LastSuccessfulTestFailoverTime),
    "Test Completed",
    "Test Not Recorded"
)
| summarize Machines = count() by TestStatus





ASRReplicatedItems
| summarize arg_max(TimeGenerated, *) by ReplicatedItemUniqueId
| project
    VMName = ReplicatedItemFriendlyName,
    LastRpoCalculatedTime,
    ReplicationStatus,
    RecoveryRegion,
    FailoverReadiness
| order by LastRpoCalculatedTime asc


ASRReplicatedItems
| summarize arg_max(TimeGenerated, *) by ReplicatedItemUniqueId
| summarize Machines = count() by ReplicationStatus
| order by Machines desc




ASRReplicatedItems
| summarize arg_max(TimeGenerated, *) by ReplicatedItemUniqueId
| extend TestStatus = iff(
    isnotempty(LastSuccessfulTestFailoverTime),
    "Test Completed",
    "Test Not Recorded"
)
| summarize Machines = count() by TestStatus






ASRReplicatedItems
| summarize arg_max(TimeGenerated, *) by ReplicatedItemUniqueId
| extend
    TestFailoverDone = isnotempty(LastSuccessfulTestFailoverTime),
    FailoverPerformed = ActiveLocation == RecoveryRegion
| extend
    HealthStatus = case(
        ProtectionInfo !~ "Protected" or isnotempty(ReplicationHealthErrors), "Critical",
        not(TestFailoverDone) or FailoverPerformed, "Warning",
        ReplicationStatus =~ "Protected" and ProtectionInfo =~ "Protected", "Healthy",
        "Warning"
    )
| extend
    StatusIcon = case(
        HealthStatus == "Healthy", "🟢",
        HealthStatus == "Warning", "🟠",
        HealthStatus == "Critical", "🔴",
        "⚪"
    )
| summarize Machines = count() by HealthStatus, StatusIcon
| order by
    case(
        HealthStatus == "Critical", 1,
        HealthStatus == "Warning", 2,
        HealthStatus == "Healthy", 3,
        4
    ) asc


