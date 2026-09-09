Get-CimInstance Win32_UserProfile |
Where-Object {$_.LocalPath -like "*Shubham235269*"} |
Select-Object LocalPath, SID, Loaded, Special


Get-ChildItem "C:\Users\Shubham235269_OLD" -Force | Select-Object Name, Mode


Remove-Item "C:\Users\Shubham235269_OLD" -Recurse -Force