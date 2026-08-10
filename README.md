VMComputer
| summarize arg_max(TimeGenerated, *) by Computer
| project
    Computer,
    Cpus = LogicalProcessorCount,
    MemoryGB = round(PhysicalMemoryMB / 1024.0, 1),
    AzureResourceId
| order by Computer asc