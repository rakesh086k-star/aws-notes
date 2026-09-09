Get-CimInstance Win32_UserProfile |
Where-Object {$_.LocalPath -like "*Shubham235269*"} |
Select-Object LocalPath, SID, Loaded, Special


Get-ChildItem "C:\Users\Shubham235269_OLD" -Force | Select-Object Name, Mode


Remove-Item "C:\Users\Shubham235269_OLD" -Recurse -Force



Stop-Process -Name Dropbox -Force -ErrorAction SilentlyContinue


takeown /F "C:\Users\Shubham235269" /R /D Y
icacls "C:\Users\Shubham235269" /grant Administrators:F /T
