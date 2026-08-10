Get-AzVM -Status |
Select-Object `
    Name,
    ResourceGroupName,
    Location,
    @{Name="VMSize";Expression={$_.HardwareProfile.VmSize}} |
Format-Table -AutoSize