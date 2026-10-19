
Resources
| where type =~ 'microsoft.compute/virtualmachines'
    or type =~ 'microsoft.compute/virtualmachinescalesets'
| extend isVMSS = type =~ 'microsoft.compute/virtualmachinescalesets'
| extend img = iff(
    isVMSS,
    properties.virtualMachineProfile.storageProfile.imageReference,
    properties.storageProfile.imageReference
)
| extend publisher = tostring(img.publisher),
         offer = tostring(img.offer),
         sku = tostring(img.sku),
         version = tostring(img.version),
         imageId = tostring(img.id)
| extend osType = iff(
    isVMSS,
    tostring(properties.virtualMachineProfile.storageProfile.osDisk.osType),
    tostring(properties.storageProfile.osDisk.osType)
)
| project subscriptionId, resourceGroup, name,
          resourceType=type, location, osType,
          publisher, offer, sku, version, imageId
| order by resourceGroup asc, name asc
