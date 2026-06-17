Get-SecureBootUEFI -Name db


Confirm-SecureBootUEFI



search *
| summarize Count=count() by $table
| sort by Count desc


union withsource=TableName *
| summarize Records=count() by TableName
| sort by Records desc

