ASRReplicatedItems
| summarize Machines = dcount(ReplicatedItemUniqueId)
    by PrimaryFabricName, RecoveryRegion
| order by Machines desc


ASRReplicatedItems
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