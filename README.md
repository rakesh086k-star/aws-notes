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




param (
    [string]$VMName,
    [string]$ResourceGroup
)

$command = @'
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v ProductName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v DisplayVersion
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v CurrentBuild
'@

$result = Invoke-AzVMRunCommand `
    -ResourceGroupName $ResourceGroup `
    -VMName $VMName `
    -CommandId 'RunPowerShellScript' `
    -ScriptString $command

$result.Value[0].Message








param (
    [string]$VMName,
    [string]$ResourceGroup
)

Connect-AzAccount -Identity

Set-AzContext -SubscriptionId "YOUR-SUBSCRIPTION-ID"

$command = @'
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v ProductName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v DisplayVersion
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v CurrentBuild
'@

$result = Invoke-AzVMRunCommand `
    -ResourceGroupName $ResourceGroup `
    -VMName $VMName `
    -CommandId 'RunPowerShellScript' `
    -ScriptString $command

$result.Value[0].Message




param (
    [string]$VMName,
    [string]$ResourceGroup
)

Disable-AzContextAutosave -Scope Process

$context = (Connect-AzAccount -Identity).Context

Set-AzContext -SubscriptionId $context.Subscription.Id -DefaultProfile $context

$command = @'
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v ProductName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v DisplayVersion
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v CurrentBuild
'@

$result = Invoke-AzVMRunCommand `
    -ResourceGroupName $ResourceGroup `
    -VMName $VMName `
    -CommandId 'RunPowerShellScript' `
    -ScriptString $command `
    -DefaultProfile $context

$result.Value[0].Message





$context = (Connect-AzAccount -Identity).Context
Set-AzContext -SubscriptionId $context.Subscription -DefaultProfile $context













param (
    [string]$VMName,
    [string]$ResourceGroup
)

# Prevent inherited Azure context issues
Disable-AzContextAutosave -Scope Process

# Login using Automation Account Managed Identity
$context = (Connect-AzAccount -Identity).Context

# Set Azure subscription context
Set-AzContext -SubscriptionId "YOUR-SUBSCRIPTION-ID" -DefaultProfile $context

# Command to fetch Windows OS details
$command = @'
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v ProductName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v DisplayVersion
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v CurrentBuild
'@

# Execute command on VM
$result = Invoke-AzVMRunCommand `
    -ResourceGroupName $ResourceGroup `
    -VMName $VMName `
    -CommandId 'RunPowerShellScript' `
    -ScriptString $command `
    -DefaultProfile $context

# Show output
$result.Value[0].Message












param (
    [string]$VMName,
    [string]$ResourceGroup
)

try {

    # Prevent Azure context conflicts
    Disable-AzContextAutosave -Scope Process

    # Login using Automation Account Managed Identity
    $context = (Connect-AzAccount -Identity).Context

    # Set Subscription Context
    Set-AzContext `
        -SubscriptionId "YOUR-SUBSCRIPTION-ID" `
        -DefaultProfile $context

    # PowerShell command to run inside VM
    $command = @'
$cv = Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion"

Write-Output "====================================="
Write-Output "VM OS VERSION DETAILS"
Write-Output "====================================="
Write-Output "OS Name      : $($cv.ProductName)"
Write-Output "Version      : $($cv.DisplayVersion)"
Write-Output "Full Build   : $($cv.CurrentBuild).$($cv.UBR)"
Write-Output "Computer Name: $env:COMPUTERNAME"
Write-Output "====================================="
'@

    # Execute command on VM
    $result = Invoke-AzVMRunCommand `
        -ResourceGroupName $ResourceGroup `
        -VMName $VMName `
        -CommandId 'RunPowerShellScript' `
        -ScriptString $command `
        -DefaultProfile $context

    # Display output
    $result.Value[0].Message

}
catch {

    Write-Output "====================================="
    Write-Output "ERROR OCCURRED"
    Write-Output "====================================="
    Write-Output $_.Exception.Message
    Write-Output "====================================="
}


# VM OS Version Checker

Enter VM Hostname and Resource Group to fetch Windows OS details.






Hi Team,

We have successfully implemented an automated solution for validating VM operating system version and build details through Azure Automation.

Previously, validating OS version/build information required manual VM access or user coordination, especially in scenarios where:
- Users were actively logged into dedicated VMs
- VM access was restricted during business hours
- Operations teams depended on user availability
- Troubleshooting and patch validation activities consumed additional time

To address this, we designed and implemented an automation-based solution using Azure Automation Runbooks and dashboard integration.

Key Highlights:
- Automated VM OS version and build retrieval
- No direct VM login required
- Reduced dependency on end users
- Faster validation during incident troubleshooting and patch verification
- Simplified operational workflow for support teams
- Centralized and reusable solution for multiple environments

Current Workflow:
1. Open the dashboard utility
2. Launch the VM OS Checker
3. Enter VM hostname and resource group
4. Retrieve OS version/build details instantly

Benefits:
- Improved operational efficiency
- Reduced manual effort
- Faster troubleshooting and validation
- Better support experience
- Secure and controlled access model

This solution can further be extended for:
- Patch compliance validation
- Reboot status checks
- Installed KB verification
- Multi-VM health validation
- Automated reporting/dashboarding

Please let us know if additional enhancements or integrations are required.

Thanks & Regards,
[Your Name]



