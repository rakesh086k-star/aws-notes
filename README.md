Connect-AzAccount
Set-AzContext -Subscription "<YOUR-SUBSCRIPTION-ID>"

$resourceGroups = @("RG-Prod","RG-Test")

foreach ($rg in $resourceGroups) {

    Write-Output "Processing RG: $rg"

    Get-AzVM -ResourceGroupName $rg | ForEach-Object {

        $vm = $_

        Write-Output "Enabling Auto Shutdown for $($vm.Name)"

        New-AzResource `
          -ResourceId "/subscriptions/$((Get-AzContext).Subscription.Id)/resourceGroups/$rg/providers/Microsoft.DevTestLab/schedules/shutdown-computevm-$($vm.Name)" `
          -Location $vm.Location `
          -Properties @{
              status = "Enabled"
              taskType = "ComputeVmShutdownTask"
              dailyRecurrence = @{ time = "1900" }
              timeZoneId = "India Standard Time"
          } `
          -Force
    }
}