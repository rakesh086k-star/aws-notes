ASRReplicatedItems
| where TimeGenerated > ago(7d)
| summarize arg_max(TimeGenerated, *) by ReplicatedItemUniqueId
| project
    TimeGenerated,
    VMName = ReplicatedItemFriendlyName,
    ReplicationStatus,
    ReplicationHealth = ReplicationHealthErrors,
    RecoveryRegion,
    PrimaryFabricName,
    RecoveryFabricName,
    PrimaryActiveLocation,
    OSFamily,
    LastHeartbeat,
    LastReplicatedItem,
    MultiVMGroupId,
    FailoverReadiness,
    DataSourceFriendlyName
| order by VMName asc


ASRReplicatedItems
| where TimeGenerated > ago(7d)
| summarize arg_max(TimeGenerated, *) by ReplicatedItemFriendlyName
| project
    TimeGenerated,
    VMName = ReplicatedItemFriendlyName,
    ReplicationStatus,
    ReplicationHealth = ReplicationHealthErrors,
    RecoveryRegion,
    PrimaryFabricName,
    RecoveryFabricName,
    PrimaryActiveLocation,
    OSFamily,
    LastHeartbeat,
    LastReplicatedItem,
    MultiVMGroupId,
    FailoverReadiness,
    DataSourceFriendlyName
| order by VMName asc
