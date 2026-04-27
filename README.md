$resourceGroups = @("RG-Prod","RG-Test")

Connect-AzAccount -Identity
$subscriptionId = (Get-AzContext).Subscription.Id

foreach ($rg in $resourceGroups) {

    $vms = Get-AzVM -ResourceGroupName $rg

    foreach ($vm in $vms) {

        $vmName = $vm.Name
        $location = $vm.Location

        $scheduleName = "shutdown-computevm-$vmName"

        $uri = "https://management.azure.com/subscriptions/$subscriptionId/resourceGroups/$rg/providers/Microsoft.DevTestLab/schedules/$scheduleName?api-version=2018-09-15"

        # Try to get existing schedule
        $existing = Invoke-AzRestMethod -Method GET -Uri $uri -ErrorAction SilentlyContinue

        if ($existing) {
            Write-Output "Schedule exists for $vmName → Enabling only"

            $body = ($existing.Content | ConvertFrom-Json)
            $body.properties.status = "Enabled"

            Invoke-AzRestMethod -Method PUT -Uri $uri -Payload ($body | ConvertTo-Json -Depth 5)
        }
        else {
            Write-Output "No schedule found for $vmName → Creating new with default time"

            $body = @{
                location = $location
                properties = @{
                    status = "Enabled"
                    taskType = "ComputeVmShutdownTask"
                    dailyRecurrence = @{
                        time = "1900"   # fallback only if not exists
                    }
                    timeZoneId = "India Standard Time"
                }
            } | ConvertTo-Json -Depth 5

            Invoke-AzRestMethod -Method PUT -Uri $uri -Payload $body
        }
    }
}