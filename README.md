RG_NAME="your-resource-group-name"

for vm in $(az vm list -g $RG_NAME --query "[].name" -o tsv); do
  echo "VM: $vm"
  az vm run-command invoke \
    --command-id RunPowerShellScript \
    --name "$vm" \
    --resource-group "$RG_NAME" \
    --scripts "Get-ComputerInfo | Select -ExpandProperty WindowsVersion" \
    --query "value[0].message" -o tsv
  echo "----------------------"
done
