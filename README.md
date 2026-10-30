az vm boot-diagnostics get-boot-log \
  --resource-group "RG-AVD-GA-EI-US" \
  --name "AZEIEUWGGA-66"


az vm show \
  --resource-group "RG-AVD-GA-EI-US" \
  --name "AZEIEUWGGA-66" \
  --query "diagnosticsProfile.bootDiagnostics" \
  --output json
