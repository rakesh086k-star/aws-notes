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