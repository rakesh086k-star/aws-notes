Connect-AzAccount -Identity

# VM Resource Group
$VMResourceGroup = "RG-AVD-Convex-EI-Uk"

# AVD Host Pool Details
$HostPoolName = "Your-HostPool-Name"
$HostPoolResourceGroup = "Your-HostPool-RG"

# VM List
$VMs = @(
    "AZEIEUWCOD-16",
    "AZEIEUWCOD-163",
    "AZEIEUWCOD-175"
)

foreach ($VM in $VMs)
{
    Write-Output "----------------------------------------"
    Write-Output "Checking VM: $VM"

    # Check VM Power State
    $VMStatus = Get-AzVM `
        -ResourceGroupName $VMResourceGroup `
        -Name $VM `
        -Status

    $PowerState = ($VMStatus.Statuses | Where-Object {
        $_.Code -like "PowerState/*"
    }).DisplayStatus

    if ($PowerState -ne "VM running")
    {
        Write-Output "$VM : VM is not running - Skipped"
        continue
    }

    try
    {
        # Find session host related to VM
        $SessionHost = Get-AzWvdSessionHost `
            -ResourceGroupName $HostPoolResourceGroup `
            -HostPoolName $HostPoolName |
            Where-Object {
                $_.Name -like "*$VM*"
            }

        if (-not $SessionHost)
        {
            Write-Output "$VM : Session Host not found - Skipped"
            continue
        }

        # Get User Sessions
        $Sessions = Get-AzWvdUserSession `
            -ResourceGroupName $HostPoolResourceGroup `
            -HostPoolName $HostPoolName `
            -SessionHostName ($SessionHost.Name.Split('/')[-1])

        if ($Sessions.Count -gt 0)
        {
            Write-Output "$VM : User Session Exists (Active or Disconnected) - Restart Skipped"

            foreach ($Session in $Sessions)
            {
                Write-Output "User: $($Session.UserPrincipalName)"
                Write-Output "Session State: $($Session.SessionState)"
            }

            continue
        }

        Write-Output "$VM : No User Session Found - Restarting VM"

        Restart-AzVM `
            -ResourceGroupName $VMResourceGroup `
            -Name $VM `
            -NoWait

        Write-Output "$VM : Restart Request Submitted"
    }
    catch
    {
        Write-Output "$VM : Error - $($_.Exception.Message)"
    }
}