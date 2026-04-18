$rgList = Get-AzResourceGroup
$locks = Get-AzResourceLock

$result = foreach ($rg in $rgList) {
    $lock = $locks | Where-Object { $_.ResourceGroupName -eq $rg.ResourceGroupName }

    [PSCustomObject]@{
        ResourceGroupName = $rg.ResourceGroupName
        LockName          = if ($lock) { $lock.Name } else { "N/A" }
        LockLevel         = if ($lock) { $lock.LockLevel } else { "No Lock" }
        Notes             = if ($lock) { $lock.Notes } else { "-" }
        SubscriptionId    = $rg.SubscriptionId
    }
}

$result | Export-Csv "C:\Temp\RG-Lock-Report.csv" -NoTypeInformation
