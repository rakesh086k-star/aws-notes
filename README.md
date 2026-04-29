az vm get-instance-view \
  --resource-group YOUR_RG_NAME \
  --name YOUR_VM_NAME \
  --query "instanceView.statuses[?starts_with(code, 'PowerState/')].displayStatus" \
  -o tsv

