Get-ChildItem C:\Windows\Temp\teamsprovision.log* |
Sort-Object LastWriteTime -Descending |
Select-Object -First 1 |
ForEach-Object {
    Select-String -Path $_.FullName `
    -Pattern '0x80070490|0x80073CF9|failed|error' `
    -Context 2,4
}