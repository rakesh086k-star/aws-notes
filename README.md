ASRReplicatedItems
| summarize TotalProtectedMachines = dcount(ReplicatedItemUniqueId)