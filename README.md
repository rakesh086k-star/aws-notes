Connect-AzAccount -Identity

$VMResourceGroup = "RG-AVD-Convex-EI-Uk"
$HostPoolRG      = "Your-HostPool-RG"
$HostPoolName    = "Your-HostPool-Name"

$VMs = @(
    "AZEIEUWCOD-16",
    "AZEIEUWCOD-163",
    "AZEIEUWCOD-175"
)

foreach ($VM in $VMs)
{
    Write-Output "================================="
    Write-Output "Checking VM: $VM"

    # Check VM Power State
    $PowerState = (
        Get-AzVM -ResourceGroupName $VMResourceGroup -Name $VM -Status
    ).Statuses |
    Where-Object {$_.Code -like "PowerState/*"} |
    Select-Object -ExpandProperty Code

    Write-Output "Power State: $PowerState"

    if ($PowerState -ne "PowerState/running")
    {
        Write-Output "$VM : VM is Stopped/Deallocated - Skipped"
        continue
    }

    # Get Session Host
    $SessionHost = Get-AzWvdSessionHost `
        -ResourceGroupName $HostPoolRG `
        -HostPoolName $HostPoolName |
        Where-Object {
            $_.Name -like "*$VM*"
        }

    if (-not $SessionHost)
    {
        Write-Output "$VM : Session Host Not Found - Restarting VM"

        Restart-AzVM `
            -ResourceGroupName $VMResourceGroup `
            -Name $VM `
            -NoWait

        continue
    }

    $SessionHostName = ($SessionHost.Name -split "/")[-1]

    # Get User Sessions
    $Sessions = Get-AzWvdUserSession `
        -ResourceGroupName $HostPoolRG `
        -HostPoolName $HostPoolName `
        -SessionHostName $SessionHostName `
        -ErrorAction SilentlyContinue

    Write-Output "Session Count: $($Sessions.Count)"

    if ($Sessions.Count -gt 0)
    {
        Write-Output "$VM : Active/Disconnected Session Found - Restart Skipped"
        continue
    }

    Write-Output "$VM : No Session Found - Restarting"

    Restart-AzVM `
        -ResourceGroupName $VMResourceGroup `
        -Name $VM `
        -NoWait

    Write-Output "$VM : Restart Request Submitted"
} 