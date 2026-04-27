# Login & context (already done by you)
# Connect-AzAccount
# Set-AzContext -Subscription "<SUBSCRIPTION-ID>"

$resourceGroups = @("RG-Prod","RG-Test")

$subscriptionId = (Get-AzContext).Subscription.Id

foreach ($rg in $resourceGroups) {

    $vms = Get-AzVM -ResourceGroupName $rg

    foreach ($vm in $vms) {

        $vmName = $vm.Name
        $scheduleName = "shutdown-computevm-$vmName"

        $uri = "https://management.azure.com/subscriptions/$subscriptionId/resourceGroups/$rg/providers/Microsoft.DevTestLab/schedules/$scheduleName?api-version=2018-09-15"

        # Get existing schedule
        $existing = Invoke-AzRestMethod -Method GET -Uri $uri -ErrorAction SilentlyContinue

        if ($existing) {
            Write-Output "Enabling Auto Shutdown for $vmName"

            $body = ($existing.Content | ConvertFrom-Json)
            $body.properties.status = "Enabled"

            Invoke-AzRestMethod -Method PUT -Uri $uri -Payload ($body | ConvertTo-Json -Depth 5)
        }
        else {
            Write-Output "No existing schedule for $vmName (skipped)"
        }
    }
}