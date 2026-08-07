ASRReplicatedItems
| summarize arg_max(TimeGenerated, *) by ReplicatedItemUniqueId
| project
    VMName = ReplicatedItemFriendlyName,
    PrimaryRegion = PrimaryFabricName,
    RecoveryRegion,
    ReplicationStatus,
    FailoverReadiness,
    ReplicationHealthErrors,
    LastHeartbeat,
    LastRpoCalculatedTime,
    LastSuccessfulTestFailoverTime
| order by PrimaryRegion asc, RecoveryRegion asc, VMName asc