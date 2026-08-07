let TotalMachines = 70;
let ProtectedMachines =
    toscalar(
        ASRReplicatedItems
        | summarize dcount(ReplicatedItemUniqueId)
    );
union
    (print Status="Protected", Count=ProtectedMachines),
    (print Status="Not Protected", Count=TotalMachines-ProtectedMachines)