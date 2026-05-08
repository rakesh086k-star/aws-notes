Dear Team,

Please find below the estimated monthly cost for the current AVD environment and Disaster Recovery (ASR) enablement.

Current Environment Configuration:
- AVD Session Hosts: 10
- VM Size: Standard_D4s_v5
- Configuration: 4 vCPU, 32 GB RAM
- Disk Type: Standard SSD Managed Disk

| Component | Configuration | Qty | Estimated Monthly Cost |
|---|---|---|---|
| AVD Session Host VM | Standard_D4s_v5 (4 vCPU, 32 GB RAM) | 10 | ~$2,200 – $2,800 |
| OS / Managed Disks | Standard SSD Managed Disk | 10 | ~$150 – $250 |
| Azure Site Recovery (ASR) | DR Protection per VM | 10 | ~$250 – $300 |
| Replica Managed Disks | Standard SSD Replica Disks | 10 | ~$150 – $250 |
| Cache Storage Account | ASR Replication Cache Storage | 1 | ~$50 – $120 |
| Replication Traffic | East US → South US | - | ~$50 – $150 |

Estimated Monthly Cost Summary:

| Environment | Estimated Cost |
|---|---|
| AVD Environment Only | ~$2,400 – $3,000/month |
| AVD + ASR Disaster Recovery | ~$3,000 – $3,900/month |

For Disaster Recovery enablement, Azure Site Recovery (ASR) will replicate VM managed disks from the primary region (East US) to the secondary region (South US). A cache storage account will also be configured to support replication and recovery operations.

Please note that the above pricing is an approximate estimate and may vary based on actual utilization and Azure consumption.

Thanks & Regards,  
[Your Name]