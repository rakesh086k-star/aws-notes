Connect-AzAccount -Identity

$ResourceGroup = "RG-AVD-Convex-EI-Uk"

$VMs = @(
    "AZEIEUWCOD-16",
    "AZEIEUWCOD-163",
    "AZEIEUWCOD-175"
)

foreach ($VM in $VMs)
{
    $VMStatus = Get-AzVM `
        -ResourceGroupName $ResourceGroup `
        -Name $VM `
        -Status

    $PowerState = ($VMStatus.Statuses | Where-Object {
        $_.Code -like "PowerState/*"
    }).DisplayStatus

    if ($PowerState -eq "VM running")
    {
        Write-Output "$VM : Already Running"
    }
    else
    {
        Start-AzVM `
            -ResourceGroupName $ResourceGroup `
            -Name $VM `
            -NoWait

        Write-Output "$VM : Start Request Submitted"
    }
}