Resources
| where type =~ 'microsoft.compute/virtualmachines'
| extend OS_Type = tostring(properties.storageProfile.osDisk.osType)
| extend Image_Publisher = tostring(properties.storageProfile.imageReference.publisher)
| extend Image_Offer = tostring(properties.storageProfile.imageReference.offer)
| extend Image_SKU = tostring(properties.storageProfile.imageReference.sku)
| extend Image_ID = tostring(properties.storageProfile.imageReference.id)
| project name, resourceGroup, location, OS_Type,
          Image_Publisher, Image_Offer, Image_SKU, Image_ID
| order by name asc