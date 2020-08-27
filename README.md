# arm-msi-function-app

ARM Template to create an Azure Function App that uses a systemAssigned identity to access a key vault for it's secrets.

This sample illustrates a dependancy issue with the systemIdentity being created prior to it being referenced by the AccessPolicy.

The App Settings that use the Key Vault Secrets is separated to allow for secrets to be added to the Key Vault out side of the ARM Deployment process.

To see this error in play. It needs to be the initiali deployment of the resources. A second run of the same template will not produce the error.

Start by creating a new resource group:

``` bash
az group create -l <location> -n <resource group name>
```

Then deploy the template

``` bash
az deployment group create -f azure-deploy.json -g <resource group name>-p groupId=<a unique to you value>
```

The question is how to make line 91 of the azure-deploy.json actually wait for the systemAssigned principal that is created by the functionApp? The function is reporting completion dispite that assignedPrinipal not being there.

``` json
"dependsOn": [
    "[resourceId('Microsoft.Web/sites', variables('functionAppName'))]"
],
```

UPDATE: solution found. The issue isn't with line 91, it was with the reference to the identity itself.

Instead of using:

```json
    "tenantId": "[reference(concat('Microsoft.Web/sites/',  variables('functionAppName'), '/providers/Microsoft.ManagedIdentity/Identities/default'), '2015-08-31-PREVIEW').tenantId]",
    "objectId": "[reference(concat('Microsoft.Web/sites/',  variables('functionAppName'), '/providers/Microsoft.ManagedIdentity/Identities/default'), '2015-08-31-PREVIEW').principalId]",
```

as documentated by: [Use Key Vault references for App Service and Azure Functions](https://docs.microsoft.com/en-us/azure/app-service/app-service-key-vault-references)

Use this:

```json
    "tenantId": "[reference(resourceId('Microsoft.Web/sites', variables('functionAppName')), '2018-02-01', 'Full').identity.tenantId]",
    "objectId": "[reference(resourceId('Microsoft.Web/sites', variables('functionAppName')), '2018-02-01', 'Full').identity.tenantId]",
```

as documented by: [Overview Managed Identity](https://docs.microsoft.com/en-us/azure/app-service/overview-managed-identity?tabs=dotnet#using-an-azure-resource-manager-template)
