az extension update --name vm-repair


az vm encryption show \
  --resource-group RG-AVD-Convex-EI-UK \
  --name AZEIEUWCOD-205 \
  --query "disks[].encryptionSettings[].enabled" \
  -o json