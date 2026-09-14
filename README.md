Get-ChildItem "\\stfslogixexperian01.file.core.windows.net\profiledata" -Include *.vhd,*.vhdx -File -Recurse -ErrorAction SilentlyContinue |
Where-Object {$_.LastWriteTime -lt (Get-Date).AddMonths(-3)} |
Select-Object FullName, LastWriteTime, @{Name="SizeGB";Expression={[math]::Round($_.Length/1GB,2)}} |
Sort-Object LastWriteTime