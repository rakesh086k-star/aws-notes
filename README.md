az vm get-instance-view \
  --resource-group RG-AVD-Convex-EI-UK \
  --name AZEIEUWCOD-205 \
  --query "instanceView.statuses[?starts_with(code, 'PowerState/')].displayStatus" \
  -o tsv