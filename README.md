$VMs = Get-AzVM -ResourceGroupName "RG-Name"

$VMs | Select-Object Name,
@{N="VMSize";E={$_.HardwareProfile.VmSize}},
Location