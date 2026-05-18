param(
    [ValidateSet("Single","All")]
    [string]$Mode,

    [string]$VMName
)

# Fixed Resource Group
$ResourceGroup = "RG-AVD-GA-EI-UW-DR"

# Login using Managed Identity
Disable-AzContextAutosave -Scope Process
$context = (Connect-AzAccount -Identity).Context

# Set Subscription Context
Set-AzContext `
-SubscriptionId "YOUR-SUBSCRIPTION-ID" `
-DefaultProfile $context

# Validation for Single VM
if ($Mode -eq "Single" -and [string]::IsNullOrWhiteSpace($VMName)) {

    Write-Output "Please provide VM Name"
    return
}

# ALL VM Mode
if ($Mode -eq "All") {

    $VMs = Get-AzVM -ResourceGroupName $ResourceGroup

    foreach ($VM in $VMs) {

        Write-Output "=================================="
        Write-Output "Checking VM: $($VM.Name)"
        Write-Output "=================================="

        $result = Invoke-AzVMRunCommand `
        -ResourceGroupName $ResourceGroup `
        -VMName $VM.Name `
        -CommandId 'RunPowerShellScript' `
        -ScriptString @'

$cv = Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion"

Write-Output "VM OS VERSION DETAILS"
Write-Output "-----------------------------"
Write-Output "OS Name       : $($cv.ProductName)"
Write-Output "Version       : $($cv.DisplayVersion)"
Write-Output "Build Number  : $($cv.CurrentBuild)"
Write-Output "Computer Name : $env:COMPUTERNAME"

'@

        $result.Value.Message
    }
}

# SINGLE VM Mode
elseif ($Mode -eq "Single") {

    Write-Output "=================================="
    Write-Output "Checking VM: $VMName"
    Write-Output "=================================="

    $result = Invoke-AzVMRunCommand `
    -ResourceGroupName $ResourceGroup `
    -VMName $VMName `
    -CommandId 'RunPowerShellScript' `
    -ScriptString @'

$cv = Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion"

Write-Output "VM OS VERSION DETAILS"
Write-Output "-----------------------------"
Write-Output "OS Name       : $($cv.ProductName)"
Write-Output "Version       : $($cv.DisplayVersion)"
Write-Output "Build Number  : $($cv.CurrentBuild)"
Write-Output "Computer Name : $env:COMPUTERNAME"

'@

    $result.Value.Message
}

else {

    Write-Output "Please select Mode as Single or All"
}



param(
    [Parameter(Mandatory=$true)]
    [ValidateSet("Single","All")]
    [string]$Mode,

    [string]$VMName
)


param(
    [Parameter(Mandatory=$true)]
    [string]$Mode,

    [string]$VMName
)




param(
    [string]$Mode,

    [string]$VMName
)

# Fixed Resource Group
$ResourceGroup = "RG-AVD-GA-EI-UW-DR"

# Login using Managed Identity
Disable-AzContextAutosave -Scope Process

$context = (Connect-AzAccount -Identity).Context

# Hide Azure Context Output
Set-AzContext `
-SubscriptionId "YOUR-SUBSCRIPTION-ID" `
-DefaultProfile $context | Out-Null

# Validate Mode
if ($Mode -notin @("Single","All")) {

    Write-Output "Please enter Mode as Single or All"
    return
}

# Validate VM Name
if ($Mode -eq "Single" -and [string]::IsNullOrWhiteSpace($VMName)) {

    Write-Output "Please provide VM Name"
    return
}

# ALL VM Mode
if ($Mode -eq "All") {

    $VMs = Get-AzVM -ResourceGroupName $ResourceGroup

    foreach ($VM in $VMs) {

        $result = Invoke-AzVMRunCommand `
        -ResourceGroupName $ResourceGroup `
        -VMName $VM.Name `
        -CommandId 'RunPowerShellScript' `
        -ScriptString @'

$cv = Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion"

Write-Output "-----------------------------------"
Write-Output "VM Name      : $env:COMPUTERNAME"
Write-Output "OS Name      : $($cv.ProductName)"
Write-Output "Version      : $($cv.DisplayVersion)"
Write-Output "Build Number : $($cv.CurrentBuild)"
Write-Output "-----------------------------------"

'@

        $result.Value.Message
    }
}

# SINGLE VM Mode
elseif ($Mode -eq "Single") {

    $result = Invoke-AzVMRunCommand `
    -ResourceGroupName $ResourceGroup `
    -VMName $VMName `
    -CommandId 'RunPowerShellScript' `
    -ScriptString @'

$cv = Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion"

Write-Output "-----------------------------------"
Write-Output "VM Name      : $env:COMPUTERNAME"
Write-Output "OS Name      : $($cv.ProductName)"
Write-Output "Version      : $($cv.DisplayVersion)"
Write-Output "Build Number : $($cv.CurrentBuild)"
Write-Output "-----------------------------------"

'@

    $result.Value.Message
}

