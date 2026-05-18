param(
    [string]$VMName,
    [string]$Mode
)

# Fixed Resource Group
$ResourceGroup = "RG-AVD-GA-EI-UW-DR"

# Login using Managed Identity
Disable-AzContextAutosave -Scope Process
$context = (Connect-AzAccount -Identity).Context

# Set Subscription
Set-AzContext `
-SubscriptionId "YOUR-SUBSCRIPTION-ID" `
-DefaultProfile $context

# If user wants all VMs
if ($Mode -eq "All") {

    $VMs = Get-AzVM -ResourceGroupName $ResourceGroup

    foreach ($VM in $VMs) {

        Write-Output "Checking VM: $($VM.Name)"

        Invoke-AzVMRunCommand `
        -ResourceGroupName $ResourceGroup `
        -VMName $VM.Name `
        -CommandId 'RunPowerShellScript' `
        -ScriptString @'
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion"
'@
    }
}

# If user wants single VM
elseif ($Mode -eq "Single") {

    Invoke-AzVMRunCommand `
    -ResourceGroupName $ResourceGroup `
    -VMName $VMName `
    -CommandId 'RunPowerShellScript' `
    -ScriptString @'
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion"
'@
}

else {
    Write-Output "Please select Mode as Single or All"
}




param(
    [ValidateSet("Single","All")]
    [string]$Mode,

    [string]$VMName
)