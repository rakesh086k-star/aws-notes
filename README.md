$resourceGroups = @("RG-Prod","RG-Test")

Connect-AzAccount -Identity
Set-AzContext -Subscription "<YOUR-SUBSCRIPTION-ID>"

foreach ($rg in $resourceGroups) {

    Write-Output "Processing RG: $rg"

    $vms = Get-AzVM -ResourceGroupName $rg

    foreach ($vm in $vms) {
        Write-Output "VM Found: $($vm.Name)"
    }
}