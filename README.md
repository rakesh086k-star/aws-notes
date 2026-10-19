
Resources
| where type =~ 'microsoft.compute/virtualmachines'
    or type =~ 'microsoft.compute/virtualmachinescalesets'
| extend img = iff(
    type =~ 'microsoft.compute/virtualmachinescalesets',
    properties.virtualMachineProfile.storageProfile.imageReference,
    properties.storageProfile.imageReference
)
| extend publisher = tostring(img.publisher),
         offer = tostring(img.offer),
         sku = tostring(img.sku),
         imageId = tostring(img.id)
| where sku =~ '19h2-evd'
    or offer =~ 'Windows-10'
    or imageId contains '19h2-evd'
| project subscriptionId, resourceGroup, name, type,
          publisher, offer, sku, imageId
| order by name asc
