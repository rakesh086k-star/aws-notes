Test-Path "\\stfslogixexperian01.file.core.windows.net\profiledata"



Get-ChildItem "\\stfslogixexperian01.file.core.windows.net\profiledata" -Include *.vhd,*.vhdx -File -Recurse -ErrorAction SilentlyContinue | Where-Object {$_.LastWriteTime -lt (Get-Date).AddDays(-120)} | Select-Object FullName,Length,LastWriteTime