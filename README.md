param(
    [string]$VMName,
    [string]$ResourceGroup
)

Connect-AzAccount -Identity

$Script = @'

Write-Output "========== DISK CLEANUP STARTED =========="

$DeletedCount = 0

$FreeBefore = [math]::Round((Get-PSDrive -Name C).Free / 1GB,2)

Write-Output "Free Space Before Cleanup : $FreeBefore GB"

$Paths = @(
"C:\Windows\Temp",
"C:\Windows\Installer",
"C:\Windows\ccmcache"
)

foreach ($Path in $Paths) {

    if (Test-Path $Path) {

        Write-Output "Cleaning Path : $Path"

        Get-ChildItem -Path $Path -Recurse -Force -ErrorAction SilentlyContinue | ForEach-Object {

            try {

                Remove-Item -Path $_.FullName -Recurse -Force -ErrorAction Stop
                $DeletedCount++

            }
            catch {

                Write-Output "Skipped File : $($_.FullName)"

            }

        }

    }

}

Get-ChildItem "C:\Users" -Directory -ErrorAction SilentlyContinue | ForEach-Object {

    $UserTemp = Join-Path $_.FullName "AppData\Local\Temp"

    if (Test-Path $UserTemp) {

        Write-Output "Cleaning User Temp : $UserTemp"

        Get-ChildItem -Path $UserTemp -Recurse -Force -ErrorAction SilentlyContinue | ForEach-Object {

            try {

                Remove-Item -Path $_.FullName -Recurse -Force -ErrorAction Stop
                $DeletedCount++

            }
            catch {

                Write-Output "Skipped File : $($_.FullName)"

            }

        }

    }

}

Start-Sleep -Seconds 5

$FreeAfter = [math]::Round((Get-PSDrive -Name C).Free / 1GB,2)

$Recovered = [math]::Round(($FreeAfter - $FreeBefore),2)

Write-Output "=========================================="
Write-Output "Cleanup Completed Successfully"
Write-Output "Deleted Files Count : $DeletedCount"
Write-Output "Free Space Before Cleanup : $FreeBefore GB"
Write-Output "Free Space After Cleanup : $FreeAfter GB"
Write-Output "Recovered Disk Space : $Recovered GB"
Write-Output "=========================================="

'@

$Result = Invoke-AzVMRunCommand `
    -ResourceGroupName $ResourceGroup `
    -VMName $VMName `
    -CommandId 'RunPowerShellScript' `
    -ScriptString $Script

$Result.Value[0].Message