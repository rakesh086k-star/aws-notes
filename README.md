ASRReplicatedItems
| summarize arg_max(TimeGenerated, *) by ReplicatedItemFriendlyName
| summarize Count = count() by ProtectionInfo
| order by Count desc