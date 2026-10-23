Get-Process |
Sort-Object CPU -Descending |
Select-Object -First 10 ProcessName, Id, CPU,
@{Name='MemoryMB';Expression={
    [math]::Round($_.WorkingSet64 / 1MB, 0)
}}