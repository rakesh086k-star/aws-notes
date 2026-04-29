az vm run-command invoke \
  --command-id RunPowerShellScript \
  --name YOUR_VM_NAME \
  --resource-group YOUR_RG_NAME \
  --scripts "reg query 'HKLM\\SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion' /v DisplayVersion" \
  --query "value[0].message" -o tsv
