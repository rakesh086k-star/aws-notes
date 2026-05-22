param([string]$VMName,[string]$ResourceGroup)

Connect-AzAccount -Identity

$Result=Invoke-AzVMRunCommand -ResourceGroupName $ResourceGroup -VMName $VMName -CommandId 'RunPowerShellScript' -ScriptString @'

Write-Output "========== DISK CLEANUP STARTED =========="

$Days=-1
$DeletedCount=0

$FreeBefore=[math]::Round((Get-PSDrive C).Free/1GB,2)

Write-Output "Free Space Before Cleanup : $FreeBefore GB"

$CleanupPaths=@(
"C:\Windows\Temp",
"C:\Windows\SoftwareDistribution\Download",
"C:\Windows\ccmcache",
"C:\Windows\Installer"
)

foreach($Path in $CleanupPaths){

if(Test-Path $Path){

Write-Output "Cleaning Path : $Path"

Get-ChildItem $Path -Recurse -Force -ErrorAction SilentlyContinue |
Where-Object {$.LastWriteTime -lt (Get-Date).AddDays($Days)} |
ForEach-Object {

try{
Remove-Item $.FullName -Recurse -Force -ErrorAction Stop
$DeletedCount++
}
catch{
Write-Output "Skipped File : $($.FullName)"
}

}

}

}

Get-ChildItem "C:\Users" -Directory -ErrorAction SilentlyContinue |
ForEach-Object {

$TempPath="$($.FullName)\AppData\Local\Temp"

if(Test-Path $TempPath){

Write-Output "Cleaning User Temp : $TempPath"

Get-ChildItem $TempPath -Recurse -Force -ErrorAction SilentlyContinue |
Where-Object {$.LastWriteTime -lt (Get-Date).AddDays($Days)} |
ForEach-Object {

try{
Remove-Item $.FullName -Recurse -Force -ErrorAction Stop
$DeletedCount++
}
catch{
Write-Output "Skipped File : $($_.FullName)"
}

}

}

}

DISM.exe /Online /Cleanup-Image /StartComponentCleanup /ResetBase /Quiet

Clear-RecycleBin -Force -ErrorAction SilentlyContinue

$FreeAfter=[math]::Round((Get-PSDrive C).Free/1GB,2)

$Recovered=[math]::Round(($FreeAfter-$FreeBefore),2)

Write-Output "=========================================="
Write-Output "Cleanup Completed Successfully"
Write-Output "Deleted Files Count : $DeletedCount"
Write-Output "Free Space Before Cleanup : $FreeBefore GB"
Write-Output "Free Space After Cleanup : $FreeAfter GB"
Write-Output "Recovered Disk Space : $Recovered GB"
Write-Output "=========================================="

'@

$Result.Value.Message