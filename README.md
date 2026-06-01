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

    if ($PowerState -ne "VM running")
    {
        Write-Output "$VM : VM Not Running - Skipped"
        continue
    }

    # User Session Check Here

    $ActiveUsers = 0

    if ($ActiveUsers -gt 0)
    {
        Write-Output "$VM : User Logged In - Restart Skipped"
    }
    else
    {
        Restart-AzVM `
            -ResourceGroupName $ResourceGroup `
            -Name $VM `
            -NoWait

        Write-Output "$VM : Restart Initiated"
    }
}