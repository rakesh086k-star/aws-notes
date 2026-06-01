Connect-AzAccount -Identity

$Sessions = Get-AzWvdUserSession `
-ResourceGroupName "RG-AVD-Convex-EI-Uk" `
-HostPoolName "AHP_Convex_UK_EI_Dedicated"

$Sessions | Select-Object Name,SessionState