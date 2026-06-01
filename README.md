Get-AzWvdUserSession `
-ResourceGroupName "RG-AVD-Convex-EI-Uk" `
-HostPoolName "AHP_Convex_UK_EI_Dedicated" |
Where-Object {
    $_.Name -like "*AZEIEUWCOD-606*"
} |
Format-Table Name,UserPrincipalName,SessionState -AutoSize