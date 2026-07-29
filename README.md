# Get all VMs
$vms = Get-AzVM

$result = foreach ($vm in $vms) {

    $extensions = Get-AzVMExtension `
        -ResourceGroupName $vm.ResourceGroupName `
        -VMName $vm.Name `
        -ErrorAction SilentlyContinue

    $ama = $extensions | Where-Object {
        $_.ExtensionType -eq "AzureMonitorWindowsAgent" -or
        $_.ExtensionType -eq "AzureMonitorLinuxAgent"
    }

    [PSCustomObject]@{
        VMName           = $vm.Name
        ResourceGroup    = $vm.ResourceGroupName
        Location         = $vm.Location
        OperatingSystem  = $vm.StorageProfile.OSDisk.OsType
        AMAInstalled     = if ($ama) { "YES" } else { "NO" }
    }
}

$result | Sort-Object AMAInstalled, VMName | Format-Table -AutoSize