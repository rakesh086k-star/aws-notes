param([string]$VMName,[string]$ResourceGroup)

Connect-AzAccount -Identity

$Result = Invoke-AzVMRunCommand -ResourceGroupName $ResourceGroup -VMName $VMName -CommandId ‘RunPowerShellScript’ -ScriptString @’

Write-Output “========== DISK CLEANUP STARTED ==========”

$Days = -1
$DeletedCount = 0

$FreeBefore = [math]::Round((Get-PSDrive C).Free / 1GB,2)

Write-Output “Free Space Before Cleanup : $FreeBefore GB”

$CleanupPaths = @(
“C:\Windows\Temp”,
“C:\Windows\SoftwareDistribution\Download”,
“C:\Windows\ccmcache”,
“C:\Windows\Installer”
)

foreach($Path in $CleanupPaths){

if(Test-Path $Path){

Write-Output “Cleaning Path : $Path”

Get-ChildItem $Path -Recurse -Force -ErrorAction SilentlyContinue |
Where-Object {`$_.LastWriteTime -lt (Get-Date).AddDays($Days)} |
ForEach-Object {

try{

Remove-Item `$_.FullName -Recurse -Force -ErrorAction Stop
$DeletedCount++

}
catch{

Write-Output “Skipped File : $($_.FullName)”

}

}

}

}

Get-ChildItem “C:\Users” -Directory -ErrorAction SilentlyContinue |
ForEach-Object {

$TempPath = “$(`$_.FullName)\AppData\Local\Temp”

if(Test-Path $TempPath){

Write-Output “Cleaning User Temp : $TempPath”

Get-ChildItem $TempPath -Recurse -Force -ErrorAction SilentlyContinue |
Where-Object {`$_.LastWriteTime -lt (Get-Date).AddDays($Days)} |
ForEach-Object {

try{

Remove-Item `$_.FullName -Recurse -Force -ErrorAction Stop
$DeletedCount++

}
catch{

Write-Output “Skipped File : $($_.FullName)”

}

}

}

}

DISM.exe /Online /Cleanup-Image /StartComponentCleanup /ResetBase /Quiet

Clear-RecycleBin -Force -ErrorAction SilentlyContinue

$FreeAfter = [math]::Round((Get-PSDrive C).Free / 1GB,2)

$Recovered = [math]::Round(($FreeAfter - $FreeBefore),2)

Write-Output “==========================================”
Write-Output “Cleanup Completed Successfully”
Write-Output “Deleted Files Count : $DeletedCount”
Write-Output “Free Space Before Cleanup : $FreeBefore GB”
Write-Output “Free Space After Cleanup : $FreeAfter GB”
Write-Output “Recovered Disk Space : $Recovered GB”
Write-Output “==========================================”

’@

$Result.Value.Message














RGName = “Your-ResourceGroup-Name”

Get-AzVM -ResourceGroupName $RGName | ForEach-Object {

$VM = $_

$OSDisk = Get-AzDisk -ResourceGroupName $RGName
-DiskName $VM.StorageProfile.OSDisk.Name

[PSCustomObject]@{
VMName = $VM.Name
VMSize = $VM.HardwareProfile.VmSize
OSDiskName = $OSDisk.Name
OSDiskSizeGB = $OSDisk.DiskSizeGB
DiskType = $OSDisk.Sku.Name
Location = $VM.Location
}

} | Format-Table -AutoSize
:::

If you also want Data Disk details:

:::writing{variant=“standard” id=“38472”}
$RGName = “Your-ResourceGroup-Name”

Get-AzVM -ResourceGroupName $RGName | ForEach-Object {

$VM = $_

$OSDisk = Get-AzDisk -ResourceGroupName $RGName
-DiskName $VM.StorageProfile.OSDisk.Name

[PSCustomObject]@{
VMName = $VM.Name
VMSize = $VM.HardwareProfile.VmSize
DiskName = $OSDisk.Name
DiskSizeGB = $OSDisk.DiskSizeGB
DiskType = $OSDisk.Sku.Name
DiskCategory = if($OSDisk.Sku.Name -match “Premium”) {“Premium SSD”} else {“Standard SSD/HDD”}
}

foreach($Disk in $VM.StorageProfile.DataDisks){

$DiskInfo = Get-AzDisk -ResourceGroupName $RGName
-DiskName $Disk.Name

[PSCustomObject]@{
VMName = $VM.Name
VMSize = $VM.HardwareProfile.VmSize
DiskName = $DiskInfo.Name
DiskSizeGB = $DiskInfo.DiskSizeGB
DiskType = $DiskInfo.Sku.Name
DiskCategory = if($DiskInfo.Sku.Name -match “Premium”) {“Premium SSD”} else {“Standard SSD/HDD”}
}

}

} | Format-Table -AutoSize
