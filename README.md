# Get all Resource Groups
$rgList = Get-AzResourceGroup

# Get all Locks
$locks = Get-AzResourceLock

# Compare and build result
$result = foreach ($rg in $rgList) {
    
    $lock = $locks | Where-Object { $_.ResourceGroupName -eq $rg.ResourceGroupName }

    [PSCustomObject]@{
        ResourceGroupName = $rg.ResourceGroupName
        LockName          = if ($lock) { $lock.Name } else { "N/A" }
        LockLevel         = if ($lock) { $lock.LockLevel } else { "No Lock" }
        Notes             = if ($lock) { $lock.Notes } else { "-" }
    }
}

# Show in table
$result | Format-Table -AutoSize


## 🛠 Skills
- AWS
- Linux
- PowerShell / Bash
- Networking Basics

## 📂 Projects
- Coming Soon...

## 📫 Contact
- Email: rakesh086.k@gmail.com
