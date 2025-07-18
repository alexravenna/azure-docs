---
title: Use Managed Identities in Azure API Management | Microsoft Docs
description: Learn how to create system-assigned and user-assigned identities in API Management by using the Azure portal, PowerShell, and Resource Manager templates. Learn about supported scenarios with managed identities.
services: api-management
author: dlepow

ms.service: azure-api-management
ms.topic: how-to
ms.date: 05/19/2025
ms.author: danlep
ms.custom: devx-track-azurepowershell

#customer intent: As an API developer, I want to create managed identities so that API Management can access other resources. 
---

# Use managed identities in Azure API Management

[!INCLUDE [api-management-availability-all-tiers](../../includes/api-management-availability-all-tiers.md)]

This article shows how to create a managed identity for an Azure API Management instance and how to use it to access other resources. A managed identity generated by Microsoft Entra ID enables API Management to easily and securely access other resources that are protected by Microsoft Entra, like Azure Key Vault. Azure manages these identities, so you don't have to provision or rotate any secrets. For more information about managed identities, see [What are managed identities for Azure resources?](../active-directory/managed-identities-azure-resources/overview.md).

You can grant two types of identities to an API Management instance:

- A *system-assigned identity* is tied to your service and is deleted if your service is deleted. The service can have only one system-assigned identity.
- A *user-assigned identity* is a standalone Azure resource that can be assigned to your service. The service can have multiple user-assigned identities.

> [!NOTE]
> Managed identities are specific to the Microsoft Entra tenant in which your Azure subscription is hosted. They don't get updated if a subscription is moved to a different directory. If a subscription is moved, you need to re-create and reconfigure the identities.  

[!INCLUDE [api-management-workspace-availability](../../includes/api-management-workspace-availability.md)]

## Create a system-assigned managed identity

### Azure portal

To set up a managed identity in the Azure portal, you create an API Management instance and then enable the feature.

1. Create an API Management instance in the portal as you normally would. Go to it in the portal.
1. In the left menu, under **Security**, select **Managed identities**.
1. On the **System assigned** tab, change the **Status** to **On**. Select **Save**.

    :::image type="content" source="./media/api-management-howto-use-managed-service-identity/enable-system-identity.png" alt-text="Screenshot that shows how to enable a system-assigned managed identity." border="true":::

### Azure PowerShell

[!INCLUDE [updated-for-az](~/reusable-content/ce-skilling/azure/includes/updated-for-az.md)]

The following steps lead you through creating an API Management instance and assigning it an identity by using Azure PowerShell.

1. If you need to, install Azure PowerShell by following the instructions in the [Azure PowerShell guide](/powershell/azure/install-azure-powershell). Then run `Connect-AzAccount` to create a connection with Azure.

1. Use the following code to create an instance with a system-assigned managed identity. For more examples of how to use Azure PowerShell with API Management, see [API Management PowerShell samples](powershell-samples.md).

    ```azurepowershell-interactive
    # Create a resource group.
    New-AzResourceGroup -Name $resourceGroupName -Location $location

    # Create an API Management Consumption SKU service.
    New-AzApiManagement -ResourceGroupName $resourceGroupName -Name consumptionskuservice -Location $location -Sku Consumption -Organization contoso -AdminEmail contoso@contoso.com -SystemAssignedIdentity
    ```

You can also update an existing instance to create the identity:

```azurepowershell-interactive
# Get an API Management instance
$apimService = Get-AzApiManagement -ResourceGroupName $resourceGroupName -Name $apiManagementName

# Update an API Management instance
Set-AzApiManagement -InputObject $apimService -SystemAssignedIdentity
```

### Azure Resource Manager (ARM) template

You can create an API Management instance with a system-assigned identity by including the following property in the ARM template resource definition:

```json
"identity" : {
    "type" : "SystemAssigned"
}
```

This property instructs Azure to create and manage the identity for your API Management instance.

For example, a complete ARM template might look like this one:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "0.9.0.0",
    "resources": [{
        "apiVersion": "2021-08-01",
        "name": "contoso",
        "type": "Microsoft.ApiManagement/service",
        "location": "[resourceGroup().location]",
        "tags": {},
        "sku": {
            "name": "Developer",
            "capacity": "1"
        },
        "properties": {
            "publisherEmail": "admin@contoso.com",
            "publisherName": "Contoso"
        },
        "identity": {
            "type": "systemAssigned"
        }
    }]
}
```

When the instance is created, it has the following additional properties:

```json
"identity": {
    "type": "SystemAssigned",
    "tenantId": "<TENANTID>",
    "principalId": "<PRINCIPALID>"
}
```

The `tenantId` property identifies which Microsoft Entra tenant the identity belongs to. The `principalId` property is a unique identifier for the instance's new identity. Within Microsoft Entra ID, the service principal has the same name that you gave to your API Management instance.

> [!NOTE]
> An API Management instance can have both system-assigned and user-assigned identities. In that scenario, the `type` property is `SystemAssigned,UserAssigned`.

## Configure Key Vault access by using a managed identity

The following configurations are required if you want to use API Management to access certificates from an Azure key vault.

[!INCLUDE [api-management-key-vault-certificate-access](../../includes/api-management-key-vault-certificate-access.md)]

[!INCLUDE [api-management-key-vault-network](../../includes/api-management-key-vault-network.md)]

## Supported scenarios that use system-assigned identity

Following are some common scenarios for using a system-assigned managed identity in Azure API Management.

### Obtain a custom TLS/SSL certificate for the API Management instance from Key Vault

You can use the system-assigned identity of an API Management instance to retrieve custom TLS/SSL certificates that are stored in Key Vault. You can then assign these certificates to custom domains in the API Management instance. Take these considerations into account:

- The content type of the secret must be *application/x-pkcs12*. For more information, see [Domain certificate options](configure-custom-domain.md?tabs=key-vault#domain-certificate-options).
- You must use the Key Vault certificate secret endpoint, which contains the secret.

> [!Important]
> If you don't provide the object version of the certificate, API Management automatically obtains any newer version of the certificate within four hours after it's updated in Key Vault.

The following example shows an ARM template that uses the system-assigned managed identity of an API Management instance to retrieve a custom domain certificate from Key Vault.

#### Prerequisites

* An API Management instance that's configured with a system-assigned managed identity. To create the instance, you can use an [Azure Quickstart Template](https://azure.microsoft.com/resources/templates/api-management-create-with-msi/).
* A Key Vault instance in the same resource group. The instance must host a certificate that will be used as a custom domain certificate in API Management.

The template contains the following steps. 

1. Update the access policies of the Key Vault instance and allow the API Management instance to obtain secrets from it.
1. Update the API Management instance by setting a custom domain name through the certificate from the Key Vault instance.

When you run the template, provide parameter values that are appropriate for your environment.

```json
{
	"$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
	"contentVersion": "1.0.0.0",
	"parameters": {
        "apiManagementServiceName": {
            "type": "string",
            "minLength": 8,
            "metadata":{
                "description": "The name of the API Management instance"
            }
        },
		"publisherEmail": {
			"type": "string",
			"minLength": 1,
			"metadata": {
				"description": "The email address of the owner of the instance"
			}
		},
		"publisherName": {
			"type": "string",
			"minLength": 1,
			"metadata": {
				"description": "The name of the owner of the instance"
			}
		},
		"sku": {
			"type": "string",
			"allowedValues": ["Developer",
			"Standard",
			"Premium"],
			"defaultValue": "Developer",
			"metadata": {
				"description": "The pricing tier of the API Management instance"
			}
		},
		"skuCount": {
			"type": "int",
			"defaultValue": 1,
			"metadata": {
				"description": "The instance size of the API Management instance"
			}
		},
        "keyVaultName": {
            "type": "string",
            "metadata": {
                "description": "The name of the key vault"
            }
        },
		"proxyCustomHostname1": {
			"type": "string",
			"metadata": {
				"description": "Gateway custom hostname 1. Example: api.contoso.com"
			}
		},
		"keyVaultIdToCertificate": {
			"type": "string",
			"metadata": {
				"description": "Reference to the key vault certificate. Example: https://contoso.vault.azure.net/secrets/contosogatewaycertificate"
			}
		}
	},
	 "variables": {
        "apimServiceIdentityResourceId": "[concat(resourceId('Microsoft.ApiManagement/service', parameters('apiManagementServiceName')),'/providers/Microsoft.ManagedIdentity/Identities/default')]"
		    },
	"resources": [ 
   {
        "apiVersion": "2021-08-01",
        "name": "[parameters('apiManagementServiceName')]",
        "type": "Microsoft.ApiManagement/service",
        "location": "[resourceGroup().location]",
        "tags": {
        },
        "sku": {
            "name": "[parameters('sku')]",
            "capacity": "[parameters('skuCount')]"
        },
        "properties": {
            "publisherEmail": "[parameters('publisherEmail')]",
            "publisherName": "[parameters('publisherName')]"
        },
        "identity": {
            "type": "systemAssigned"
        }
    },
    {
        "type": "Microsoft.KeyVault/vaults/accessPolicies",
        "name": "[concat(parameters('keyVaultName'), '/add')]",
        "apiVersion": "2018-02-14",
        "properties": {
            "accessPolicies": [{
                "tenantId": "[reference(variables('apimServiceIdentityResourceId'), '2018-11-30').tenantId]",
                "objectId": "[reference(variables('apimServiceIdentityResourceId'), '2018-11-30').principalId]",
                "permissions": {
                     "secrets": ["get", "list"]
                }
            }]
        }
    },
	{
        "apiVersion": "2021-04-01",
		"type": "Microsoft.Resources/deployments",
        "name": "apimWithKeyVault",
		 "dependsOn": [
        "[resourceId('Microsoft.ApiManagement/service', parameters('apiManagementServiceName'))]"
        ],
        "properties": {
            "mode": "incremental",
            "template": {
                "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
				"contentVersion": "1.0.0.0",
				"parameters": {},			
				"resources": [{
					"apiVersion": "2021-08-01",
					"name": "[parameters('apiManagementServiceName')]",
					"type": "Microsoft.ApiManagement/service",
					"location": "[resourceGroup().location]",
					"tags": {
					},
					"sku": {
						"name": "[parameters('sku')]",
						"capacity": "[parameters('skuCount')]"
					},
					"properties": {
						"publisherEmail": "[parameters('publisherEmail')]",
						"publisherName": "[parameters('publisherName')]",
						"hostnameConfigurations": [{
							"type": "Proxy",
							"hostName": "[parameters('proxyCustomHostname1')]",
							"keyVaultId": "[parameters('keyVaultIdToCertificate')]"
						}]
					},
					"identity": {
						"type": "systemAssigned"
					}
				}]
		}
		}
	}
]
}
```

### Store and manage named values from Key Vault

You can use a system-assigned managed identity to access Key Vault to store and manage secrets for use in API Management policies. For more information, see [Use named values in Azure API Management policies](api-management-howto-properties.md). 

### Authenticate to a backend by using an API Management identity

You can use the system-assigned identity to authenticate to a backend service via the [authentication-managed-identity](authentication-managed-identity-policy.md) policy.

### Connect to Azure resources behind an IP firewall by using a system-assigned managed identity


API Management is a trusted Microsoft service to the following resources. This trusted status enables the service to connect to the following resources behind a firewall. After you explicitly assign the appropriate Azure role to the [system-assigned managed identity](../active-directory/managed-identities-azure-resources/overview.md) for a resource instance, the scope of access for the instance corresponds to the Azure role that's assigned to the managed identity.


- [Trusted access for Key Vault](/azure/key-vault/general/overview-vnet-service-endpoints#trusted-services)
- [Trusted access for Azure Storage](../storage/common/storage-network-security-trusted-azure-services.md?tabs=azure-portal#trusted-access-based-on-system-assigned-managed-identity)
- [Trusted access for Azure Services Bus](../service-bus-messaging/service-bus-ip-filtering.md#trusted-microsoft-services)
- [Trusted access for Azure Event Hubs](../event-hubs/event-hubs-ip-filtering.md#trusted-microsoft-services)

### Log events to an event hub

You can configure and use a system-assigned managed identity to access an event hub to log events from an API Management instance. For more information, see [How to log events to Event Hubs in Azure API Management](api-management-howto-log-event-hubs.md).

## Create a user-assigned managed identity

> [!NOTE]
> You can associate an API Management instance with as many as 10 user-assigned managed identities.

### Azure portal

To set up a managed identity in the portal, you must first create an API Management instance and [create a user-assigned identity](../active-directory/managed-identities-azure-resources/how-manage-user-assigned-managed-identities.md). Then complete the following steps.

1. Go to your API Management instance in the portal.
1. In the left menu, under **Security**, select **Managed identities**.
1. On the **User assigned** tab, select **Add**.
1. Search for the identity that you created earlier and select it. Select **Add**.

   :::image type="content" source="./media/api-management-howto-use-managed-service-identity/enable-user-assigned-identity.png" alt-text="Screenshot that shows how to enable a user-assigned managed identity." border="true" lightbox="./media/api-management-howto-use-managed-service-identity/enable-user-assigned-identity.png":::

### Azure PowerShell

[!INCLUDE [updated-for-az](~/reusable-content/ce-skilling/azure/includes/updated-for-az.md)]

The following steps lead you through creating an API Management instance and assigning it an identity by using Azure PowerShell.

1. If you need to, install Azure PowerShell by following the instructions in the [Azure PowerShell guide](/powershell/azure/install-azure-powershell). Then run `Connect-AzAccount` to create a connection with Azure.

1. Use the following code to create the instance. For more examples of how to use Azure PowerShell with API Management, see [API Management PowerShell samples](powershell-samples.md).

    ```azurepowershell-interactive
    # Create a resource group.
    New-AzResourceGroup -Name $resourceGroupName -Location $location

    # Create a user-assigned identity. This code requires installation of the Az.ManagedServiceIdentity module.
    $userAssignedIdentity = New-AzUserAssignedIdentity -Name $userAssignedIdentityName -ResourceGroupName $resourceGroupName

    # Create an API Management Consumption SKU service.
    $userIdentities = @($userAssignedIdentity.Id)

    New-AzApiManagement -ResourceGroupName $resourceGroupName -Location $location -Name $apiManagementName -Organization contoso -AdminEmail admin@contoso.com -Sku Consumption -UserAssignedIdentity $userIdentities
    ```

You can also update an existing service to assign an identity to the service:

```azurepowershell-interactive
# Get an API Management instance.
$apimService = Get-AzApiManagement -ResourceGroupName $resourceGroupName -Name $apiManagementName

# Create a user-assigned identity. This code requires installation of the Az.ManagedServiceIdentity module.
$userAssignedIdentity = New-AzUserAssignedIdentity -Name $userAssignedIdentityName -ResourceGroupName $resourceGroupName

# Update the API Management instance.
$userIdentities = @($userAssignedIdentity.Id)
Set-AzApiManagement -InputObject $apimService -UserAssignedIdentity $userIdentities
```

### ARM template

You can create an API Management instance that has an identity by including the following property in the resource definition:

```json
"identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
        "<RESOURCEID>": {}
    }
}
```

Adding the user-assigned type informs Azure to use the user-assigned identity that's specified for your instance.

For example, a complete ARM template might look like this one:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "0.9.0.0",
    "resources": [{
        "apiVersion": "2021-08-01",
        "name": "contoso",
        "type": "Microsoft.ApiManagement/service",
        "location": "[resourceGroup().location]",
        "tags": {},
        "sku": {
            "name": "Developer",
            "capacity": "1"
        },
        "properties": {
            "publisherEmail": "admin@contoso.com",
            "publisherName": "Contoso"
        },
        "identity": {
            "type": "UserAssigned",
             "userAssignedIdentities": {
                "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('identityName'))]": {}
             }
        },
         "dependsOn": [
          "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('identityName'))]"
        ]
    }]
}
```

When the service is created, it has the following additional properties:

```json
"identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
        "<RESOURCEID>": {
            "principalId": "<PRINCIPALID>",
            "clientId": "<CLIENTID>"
        }
    }
}
```

The `principalId` property is a unique identifier for the identity that's used for Microsoft Entra administration. The `clientId` property is a unique identifier for the application's new identity that's used for specifying which identity to use during runtime calls.

> [!NOTE]
> An API Management instance can have both system-assigned and user-assigned identities. In that scenario, the `type` property would be `SystemAssigned,UserAssigned`.

## Supported scenarios that use user-assigned managed identities

Following are some common scenarios for using a user-assigned managed identity in Azure API Management.

### Obtain a custom TLS/SSL certificate for the API Management instance from Key Vault

You can use a user-assigned identity to establish trust between an API Management instance and Key Vault. This trust can then be used to retrieve custom TLS/SSL certificates that are stored in Key Vault. You can then assign these certificates to custom domains in the API Management instance.

> [!IMPORTANT]
> If [Key Vault firewall](/azure/key-vault/general/network-security) is enabled on your key vault, you can't use a user-assigned identity for access from API Management. You can use the system-assigned identity instead. In Key Vault firewall, the **Allow Trusted Microsoft Services to bypass this firewall** option must be enabled.

Take these considerations into account:

- The content type of the secret must be *application/x-pkcs12*.
- You must use the Key Vault certificate secret endpoint, which contains the secret.

> [!Important]
> If you don't provide the object version of the certificate, API Management automatically obtains any newer version of the certificate within four hours after it's updated in Key Vault.


### Store and manage named values from Key Vault

You can use a user-assigned managed identity to access Key Vault to store and manage secrets for use in API Management policies. For more information, see [Use named values in Azure API Management policies](api-management-howto-properties.md). 

> [!NOTE]
> If [Key Vault firewall](/azure/key-vault/general/network-security) is enabled on your key vault, you can't use a user-assigned identity for access from API Management. You can use the system-assigned identity instead. In Key Vault firewall, the **Allow Trusted Microsoft Services to bypass this firewall** option must be enabled.

### Authenticate to a backend by using a user-assigned identity

You can use the user-assigned identity to authenticate to a backend service via the [authentication-managed-identity](authentication-managed-identity-policy.md) policy.

### Log events to an event hub

You can configure and use a user-assigned managed identity to access an event hub to log events from an API Management instance. For more information, see [How to log events to Azure Event Hubs in Azure API Management](api-management-howto-log-event-hubs.md).

## Remove an identity

You can remove a system-assigned identity by disabling the feature via the portal or an ARM template in the same way that it was created. User-assigned identities can be removed individually. To remove all identities, set the identity type to `"None"`.

Removing a system-assigned identity in this way also deletes it from Microsoft Entra ID. System-assigned identities are also automatically removed from Microsoft Entra ID when the API Management instance is deleted.

To remove all identities by using an ARM template, update this section:

```json
"identity": {
    "type": "None"
}
```

> [!Important]
> If an API Management instance is configured with a custom SSL certificate from Key Vault and you try to disable a managed identity, the request fails.
>
> You can resolve this by switching from a Key Vault certificate to an inline-encoded certificate and then disabling the managed identity. For more information, see [Configure a custom domain name](configure-custom-domain.md).

## Related content

* [What are managed identities for Azure resources?](../active-directory/managed-identities-azure-resources/overview.md)
* [Azure Resource Manager templates](https://github.com/Azure/azure-quickstart-templates)
* [Authenticate with a managed identity in a policy](authentication-managed-identity-policy.md)
