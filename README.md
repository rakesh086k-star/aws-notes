Get-AppxLog | Where-Object { $_.Id -in 401,404 } |
Select-Object -Last 5 Time, Id, Message