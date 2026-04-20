echo "ResourceGroup,LockName,LockLevel,Notes" > locks.csv

az group list --query "[].name" -o tsv | while read rg; do
    lock_count=$(az lock list --resource-group $rg --query "length(@)")

    if [ "$lock_count" -eq 0 ]; then
        echo "$rg,No Lock,None," >> locks.csv
    else
        az lock list --resource-group $rg \
        --query "[].join(',', ['${rg}', name, level, notes])" \
        -o tsv >> locks.csv
    fi
done