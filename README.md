$resourceGroups = @("RG-Prod","RG-Test")

Connect-AzAccount
Set-AzContext -Subscription "<YOUR-SUBSCRIPTION-ID>"

$subscriptionId = (Get-AzContext).Subscription.Id

foreach ($rg in $resourceGroups) {

    $vms = Get-AzVM -ResourceGroupName $rg

    foreach ($vm in $vms) {

        $vmName = $vm.Name
        $location = $vm.Location
        $scheduleName = "shutdown-computevm-$vmName"

        $uri = "https://management.azure.com/subscriptions/$subscriptionId/resourceGroups/$rg/providers/Microsoft.DevTestLab/schedules/$scheduleName?api-version=2018-09-15"

        # Get existing schedule
        $existing = Invoke-AzRestMethod -Method GET -Uri $uri -ErrorAction SilentlyContinue

        if ($existing) {

            $existingObj = $existing.Content | ConvertFrom-Json

            $time = $existingObj.properties.dailyRecurrence.time
            $timezone = $existingObj.properties.timeZoneId

            Write-Output "Re-enabling Auto Shutdown for $vmName"

            $body = @{
                location = $location
                properties = @{
                    status = "Enabled"
                    taskType = "ComputeVmShutdownTask"
                    dailyRecurrence = @{
                        time = $time
                    }
                    timeZoneId = $timezone
                    notificationSettings = @{
                        status = "Disabled"
                        timeInMinutes = 30
                    }
                }
            } | ConvertTo-Json -Depth 6

            Invoke-AzRestMethod -Method PUT -Uri $uri -Payload $body
        }
        else {
            Write-Output "No existing schedule for $vmName"
        }
    }
}