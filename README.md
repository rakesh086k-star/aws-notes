Get-ChildItem 'HKLM:\SOFTWARE\Classes\Installer\Products' |
ForEach-Object {
    $p = Get-ItemProperty $_.PSPath -ErrorAction SilentlyContinue
    if ($p.ProductName -like '*Slack*') {
        [PSCustomObject]@{
            ProductName = $p.ProductName
            RegistryKey = $_.PSChildName
            LocalPackage = $p.LocalPackage
        }
    }
} | Format-List