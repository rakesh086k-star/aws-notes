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
        Get-AzVM -ResourceGroupName $VMResourceGroup -Name $VM -Status
    ).Statuses |
    Where-Object { $_.Code -eq "PowerState/running" }

    if (-not $PowerState)
    {
        Write-Output "$VM : VM Stopped - Skipped"
        continue
    }

    # Check Active/Disconnected Sessions
    $Session = Get-AzWvdUserSession `
        -ResourceGroupName $VMResourceGroup `
        -HostPoolName $HostPoolName |
        Where-Object {
            $_.Name -match $VM
        }

    if ($Session)
    {
        Write-Output "$VM : Session Found [$($Session.SessionState)] - Skipped"
        continue
    }

    # No Session Found
    Write-Output "$VM : No Session Found - Restarting"

    Restart-AzVM `
        -ResourceGroupName $VMResourceGroup `
        -Name $VM `
        -NoWait

    Write-Output "$VM : Restart Request Submitted"
}