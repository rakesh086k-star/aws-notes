Get-AppxLog | Where-Object { $_.EventId -eq 404 } |
Select-Object -Last 5 Time, Id, Message | Format-List