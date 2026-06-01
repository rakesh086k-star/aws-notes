Connect-AzAccount -Identity

$ResourceGroup = "RG-AVD"

$VMs = @(
    "AVD-VM01",
    "AVD-VM02",
    "AVD-VM03"
)

foreach ($VM in $VMs)
{
    Start-AzVM -ResourceGroupName $ResourceGroup -Name $VM
}