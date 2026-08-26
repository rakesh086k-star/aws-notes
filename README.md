az extension add --name desktopvirtualization



az desktopvirtualization hostpool list --query "[].{Name:name,ResourceGroup:resourceGroup,Location:location,ResourceId:id}" -o table
