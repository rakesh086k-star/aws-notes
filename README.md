Get-Module -ListAvailable ActiveDirectory


Get-ADObject -LDAPFilter "(&(servicePrincipalName=*.file.core.windows.net)(!(msDS-SupportedEncryptionTypes=*)))" -Properties servicePrincipalName,msDS-SupportedEncryptionTypes | Select-Object Name,ObjectClass,servicePrincipalName,msDS-SupportedEncryptionTypes


