Connect-AzAccount

$vms = Get-AzVM -Status

$result = foreach ($vm in $vms) {

    $extensions = Get-AzVMExtension `
        -ResourceGroupName $vm.ResourceGroupName `
        -VMName $vm.Name `
        -ErrorAction SilentlyContinue

    $amaInstalled = $extensions | Where-Object {
        $_.ExtensionType -eq "AzureMonitorWindowsAgent"
    }

    [PSCustomObject]@{
        VMName         = $vm.Name
        ResourceGroup  = $vm.ResourceGroupName
        Location       = $vm.Location
        AMAInstalled   = if ($amaInstalled) { "YES" } else { "NO" }
    }
}

$result | Format-Table -AutoSize


1. Disk Space Cleanup Automation

Automated disk cleanup process for AVD session hosts to remove temporary and unwanted files regularly.

Benefits:
	•	Reduces low disk space issues
	•	Improves VM performance and stability
	•	Minimizes manual maintenance effort
	•	Helps prevent user slowness complaints

⸻

2. OS Version Checker Automation

Automated validation of OS versions across AVD session hosts to ensure compliance and consistency.

Benefits:
	•	Identifies outdated OS versions quickly
	•	Improves security and compliance tracking
	•	Reduces manual verification effort
	•	Ensures standardized environment management

⸻

3. Consolidated AVD Dashboard

Centralized dashboard to monitor disk utilization and resource status across all AVD hosts.

Benefits:
	•	Provides real-time visibility of disk usage
	•	Helps identify performance bottlenecks early
	•	Simplifies monitoring and reporting
	•	Supports proactive issue resolution

⸻

4. Health Status Validation

Automated health validation process to check overall AVD host availability and operational status.

Benefits:
	•	Detects unhealthy session hosts quickly
	•	Improves service reliability and uptime
	•	Reduces troubleshooting time
	•	Enables proactive infrastructure management




Monitoring Agent Auto-Deployment via Azure Policy

Implemented Azure Policy-based automation to automatically install the monitoring agent on all newly created AVD session hosts.

Benefits:
	•	Ensures consistent monitoring across all VMs
	•	Eliminates manual agent installation effort
	•	Improves compliance and monitoring coverage
	•	Enables faster onboarding of new session hosts

