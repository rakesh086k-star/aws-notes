ASRReplicatedItems
| summarize Machines = dcount(ReplicatedItemUniqueId)
    by PrimaryFabricName, RecoveryRegion
| order by Machines desc