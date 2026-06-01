Connect-AzAccount -Identity

# VM Resource Group
$VMResourceGroup = "RG-AVD-Convex-EI-Uk"

# Host Pool Details
$HostPoolRG = "Your-HostPool-RG"
$HostPoolName = "Your-HostPool-Name"

# VM List
$VMs = @(
    "AZEIEUWCOD-16",
    "AZEIEUWCOD-163",
    "AZEIEUWCOD-175"
)

foreach ($VM in $VMs)
{
    Write-Output "-------------------------------------"
    Write-Output "Checking VM: $VM"

    try
    {
        # Check Power State
        $VMStatus = Get-AzVM `
            -ResourceGroupName $VMResourceGroup `
            -Name $VM `
            -Status

        $PowerState = ($VMStatus.Statuses |
            Where-Object {$_.Code -like "PowerState/*"} |
            Select-Object -ExpandProperty Code)

        Write-Output "Power State: $PowerState"

        if ($PowerState -ne "PowerState/running")
        {
            Write-Output "$VM : VM is Stopped/Deallocated - Skipped"
            continue
        }

        # Find Session Host
        $SessionHost = Get-AzWvdSessionHost `
            -ResourceGroupName $HostPoolRG `
            -HostPoolName $HostPoolName |
            Where-Object {
                $_.Name -match $VM
            }

        if (-not $SessionHost)
        {
            Write-Output "$VM : Session Host not found - Skipped"
            continue
        }

        $SessionHostName = $SessionHost.Name.Split("/")[-1]

        # Get User Sessions
        $Sessions = Get-AzWvdUserSession `
            -ResourceGroupName $HostPoolRG `
            -HostPoolName $HostPoolName `
            -SessionHostName $SessionHostName `
            -ErrorAction SilentlyContinue

        if ($Sessions -and $Sessions.Count -gt 0)
        {
            Write-Output "$VM : User Session Found - Restart Skipped"

            foreach ($Session in $Sessions)
            {
                Write-Output "User: $($Session.UserPrincipalName)"
                Write-Output "State: $($Session.SessionState)"
            }

            continue
        }

        Write-Output "$VM : No User Session Found"

        Restart-AzVM `
            -ResourceGroupName $VMResourceGroup `
            -Name $VM `
            -NoWait

        Write-Output "$VM : Restart Request Submitted"
    }
    catch
    {
        Write-Output "$VM : Error"
        Write-Output $_.Exception.Message
    }
}