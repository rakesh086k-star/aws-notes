Perf
| where TimeGenerated > ago(1d)
| where (ObjectName == "Logical Disk" or ObjectName == "LogicalDisk")
  and CounterName contains "% Free Space"
  and InstanceName != "_Total"
  and InstanceName != "HarddiskVolume1"
| extend AVDHostName = Computer
| extend FreeSpace = CounterValue
| extend DriveName = InstanceName
| extend FreeSpaceGB = FreeSpace  // agar already GB hai, warna convert karo
| extend SpaceStatus = case(
    FreeSpaceGB < 5, "Critical (<5GB)",
    FreeSpaceGB < 10, "Warning (<10GB)",
    FreeSpaceGB < 20, "Low (<20GB)",
    "Healthy (>20GB)"
)
| project AVDHostName, DriveName, FreeSpaceGB, SpaceStatus, TimeGenerated
| summarize arg_max(TimeGenerated, *) by AVDHostName
| order by FreeSpaceGB asc
