Get-ChildItem "C:\ProgramData\FSLogix\Logs\Profile" |
Sort-Object LastWriteTime -Descending |
Select-Object -First 5 Name,Length,LastWriteTime