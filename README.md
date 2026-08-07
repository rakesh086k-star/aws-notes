ASRReplicatedItems
| summarize
    TotalRecords = count(),
    UniqueVMs = dcount(ReplicatedItemFriendlyName),
    UniqueIDs = dcount(ReplicatedItemUniqueId)