Get-CimInstance Win32_UserProfile |
Where-Object {$_.LocalPath -like "*Shubham235269*"} |
Select-Object LocalPath, SID, Loaded, Special