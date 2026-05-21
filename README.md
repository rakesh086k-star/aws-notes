param (
    [string]$VMName,
    [string]$ResourceGroup
)

Connect-AzAccount -Identity

$ScriptBlock = @'

$Days = -10

$Paths = @(
    "C:\Windows\Installer",
    "C:\Windows\Temp"
)

foreach ($Path in $Paths) {

    if (Test-Path $Path) {

        Get-ChildItem $Path -Recurse -Force -ErrorAction SilentlyContinue |
        Where-Object { $_.LastWriteTime -lt (Get-Date).AddDays($Days) } |
        ForEach-Object {

            try {
                Remove-Item $_.FullName -Recurse -Force -ErrorAction Stop
            }
            catch {}
        }
    }
}

Get-ChildItem "C:\Users" -Directory -ErrorAction SilentlyContinue | ForEach-Object {

    $UserTemp = "$($_.FullName)\AppData\Local\Temp"

    if (Test-Path $UserTemp) {

        Get-ChildItem $UserTemp -Recurse -Force -ErrorAction SilentlyContinue |
        Where-Object { $_.LastWriteTime -lt (Get-Date).AddDays($Days) } |
        ForEach-Object {

            try {
                Remove-Item $_.FullName -Recurse -Force -ErrorAction Stop
            }
            catch {}
        }
    }
}

DISM.exe /Online /Cleanup-Image /StartComponentCleanup /ResetBase /Quiet

Get-PSDrive C

'@

Invoke-AzVMRunCommand `
    -ResourceGroupName $ResourceGroup `
    -VMName $VMName `
    -CommandId "RunPowerShellScript" `
    -ScriptString $ScriptBlock