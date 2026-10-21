Get-ChildItem C:\Windows\Temp -Filter "teamsprovision.log*" -ErrorAction SilentlyContinue |
Select-Object Name, FullName, LastWriteTime