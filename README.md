Connect-AzAccount -Identity

$ResourceGroup = "RG-AVD-Convex-EI-Uk"

$VMs = @(
    "AZEIEUWCOD-16",
    "AZEIEUWCOD-163",
    "AZEIEUWCOD-175"
)

foreach ($VM in $VMs)
{
    Start-AzVM `
        -ResourceGroupName $ResourceGroup `
        -Name $VM `
        -NoWait

    Write-Output "Start request submitted for $VM"
}