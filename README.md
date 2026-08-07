ASRReplicatedItems
| summarize arg_max(TimeGenerated, *) by ReplicatedItemFriendlyName
| project
    TimeGenerated,
    VMName = ReplicatedItemFriendlyName,
    ReplicationStatus,
    ReplicationHealthErrors,
    RecoveryRegion,
    PrimaryFabricName,
    RecoveryFabricName,
    OSFamily,
    LastHeartbeat,
    LastReplicatedItem,
    MultiVMGroupId,
    DataSourceFriendlyName
| order by VMName asc