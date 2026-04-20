echo "ResourceGroup | LockName | LockLevel | Notes"

az group list --query "[].name" -o tsv | while read rg; do
    lock_count=$(az lock list --resource-group $rg --query "length(@)")

    if [ "$lock_count" -eq 0 ]; then
        echo "$rg | No Lock | None |"
    else
        az lock list --resource-group $rg \
        --query "[].join(' | ', ['${rg}', name, level, notes])" \
        -o tsv
    fi
done
