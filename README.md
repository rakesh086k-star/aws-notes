az vm show \
  --resource-group RG-AVD-Convex-EI-UK \
  --name AZEIEUWCOD-205 \
  --query "{Name:name,Location:location,PowerState:powerState,OSDisk:storageProfile.osDisk.name,DiskID:storageProfile.osDisk.managedDisk.id}" \
  -o json