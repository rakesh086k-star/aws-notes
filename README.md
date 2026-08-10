resources
| where type =~ 'microsoft.storage/storageaccounts'
| where properties.azureFilesIdentityBasedAuthentication.directoryServiceOptions == 'AD'
| project
    subscriptionId,
    resourceGroup,
    storageAccount=name,
    domainName=properties.azureFilesIdentityBasedAuthentication.activeDirectoryProperties.domainName,
    forestName=properties.azureFilesIdentityBasedAuthentication.activeDirectoryProperties.forestName,
    samAccountName=properties.azureFilesIdentityBasedAuthentication.activeDirectoryProperties.samAccountName,
    accountType=properties.azureFilesIdentityBasedAuthentication.activeDirectoryProperties.accountType
| order by domainName, storageAccount