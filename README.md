Connect-AzAccount -Identity

$VMResourceGroup = "RG-AVD-Convex-EI-Uk"
$HostPoolName = "AHP_Convex_UK_EI_Dedicated"

$VMs = @(
    "AZEIEUWCOD-16",
    "AZEIEUWCOD-163",
    "AZEIEUWCOD-175",
    "AZEIEUWCOD-606"
)

foreach ($VM in $VMs)
{
    # Check VM Power State
    $PowerState = (
        Get-AzVM `
        -ResourceGroupName $VMResourceGroup `
        -Name $VM `
        -Status
    ).Statuses |
    Where-Object { $_.Code -like "PowerState/*" } |
    Select-Object -ExpandProperty Code

    if ($PowerState -ne "PowerState/running")
    {
        Write-Output "$VM : VM Stopped - Skipped"
        continue
    }

    # Check Active/Disconnected Session
    $SessionExists = Get-AzWvdUserSession `
        -ResourceGroupName $VMResourceGroup `
        -HostPoolName $HostPoolName |
        Where-Object {
            $_.Name -like "*$VM*"
        }

    if ($SessionExists)
    {
        Write-Output "$VM : User Session Found (Active/Disconnected) - Skipped"
        continue
    }

    # No Session Found -> Restart VM
    Write-Output "$VM : No User Session Found - Restarting"

    Restart-AzVM `
        -ResourceGroupName $VMResourceGroup `
        -Name $VM `
        -NoWait

    Write-Output "$VM : Restart Request Submitted"
}