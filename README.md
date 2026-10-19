Resources
| where type =~ "microsoft.compute/virtualmachines"
| extend OS = tostring(properties.storageProfile.osDisk.osType)
| extend ImageId = tostring(properties.storageProfile.imageReference.id)
| extend Publisher = tostring(properties.storageProfile.imageReference.publisher)
| extend Offer = tostring(properties.storageProfile.imageReference.offer)
| extend SKU = tostring(properties.storageProfile.imageReference.sku)
| project subscriptionId, resourceGroup, name, location,
          OS, Publisher, Offer, SKU, ImageId
| order by name asc