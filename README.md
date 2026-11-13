Get-ItemProperty `
'HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*',
'HKLM:\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*',
'HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*' |
Where-Object {$_.DisplayName -like '*Slack*'} |
Select-Object DisplayName, UninstallString, PSChildName