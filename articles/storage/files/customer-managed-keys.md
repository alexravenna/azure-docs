---
title: Configure Customer-Managed Keys for Azure Files
description: Learn how to configure customer-managed keys (CMK) in Azure Key Vault for encrypting Azure file share data at rest.
author: khdownie
ms.service: azure-file-storage
ms.topic: how-to
ms.date: 05/05/2026
ms.author: kendownie
# Customer intent: "As a cloud admin, I want to learn how to configure customer-managed keys instead of Microsoft-managed keys for encrypting Azure Files data at rest."
---

# Configure customer-managed keys for Azure Files encryption

:heavy_check_mark: **Applies to:** Classic SMB and NFS file shares created with the Microsoft.Storage resource provider

:heavy_multiplication_x: **Doesn't apply to:** File shares created with the Microsoft.FileShares resource provider (preview)

Azure encrypts all data in a storage account at rest, including Azure Files data, using AES-256 encryption. By default, Microsoft manages the encryption keys for a storage account. For more control over encryption keys, you can use customer-managed keys (CMK) instead of Microsoft-managed keys to protect and control access to the encryption key that encrypts your data. This article explains how to configure customer-managed keys for Azure Files workloads.

When you configure customer-managed keys for a storage account, Azure Files data in that storage account is automatically encrypted using the customer key. No per-share opt-in is required. Customer-managed keys are stored in [Azure Key Vault](/azure/key-vault/general/overview).

## Step 1: Create or configure a Key Vault

To enable customer managed keys, you need an Azure Key Vault with purge protection enabled. You can use an existing Key Vault or create a new one. The storage account and Key Vault can be in different regions or subscriptions within the same Microsoft Entra tenant.

These instructions are for Azure Key Vault. The steps and commands for [Azure Key Vault Managed HSM](/azure/key-vault/managed-hsm/overview) (Hardware Security Module) might be different.

### [Azure portal](#tab/portal)

To create a new Key Vault by using the Azure portal, follow these steps:

1. In the Azure portal, search for **Key Vaults** and select **Create**.
1. Fill in the required fields (subscription, resource group, name, region).
1. Under **Recovery options**, select **Enable purge protection**.
1. Select **Review + create**, and then select **Create**.

For an existing key vault:

1. Go to your Key Vault in the Azure portal.
1. From the service menu, under **Settings**, select **Properties**.
1. In the **Purge protection** section, select **Enable purge protection** and then select **Save**.
1. Make sure soft delete is enabled. It's enabled by default for new Key Vaults.

### [PowerShell](#tab/powershell)

To create a new Key Vault with purge protection enabled, run the following cmdlet. Replace `<key-vault-name>`, `<resource-group>`, and `<region>` with your own values.

```azurepowershell
New-AzKeyVault `
    -Name <key-vault-name> `
    -ResourceGroupName <resource-group> `
    -Location <region> `
    -EnablePurgeProtection
```

To enable purge protection on an existing Key Vault, run the following cmdlet. Replace `<key-vault-name>` and `<resource-group>` with your own values.

```azurepowershell
Update-AzKeyVault `
    -VaultName <key-vault-name> `
    -ResourceGroupName <resource-group> `
    -EnablePurgeProtection
```

### [Azure CLI](#tab/cli)

To create a new Key Vault with purge protection enabled, run the following command. Replace `<key-vault-name>`, `<resource-group>`, and `<region>` with your own values.

```azurecli
az keyvault create \
    --name <key-vault-name> \
    --resource-group <resource-group> \
    --location <region> \
    --enable-purge-protection true
```

To enable purge protection on an existing Key Vault, run the following command. Replace `<key-vault-name>` and `<resource-group>` with your own values.

```azurecli
az keyvault update \
    --name <key-vault-name> \
    --resource-group <resource-group> \
    --enable-purge-protection true
```

---

### Assign the Key Vault Crypto Officer role

To create and manage keys in your Key Vault, you need the **Key Vault Crypto Officer** role on the Key Vault. You can assign this role to yourself by using the Azure portal, PowerShell, or Azure CLI. If you already have this role on the Key Vault, you can skip this section and proceed to Step 2.

You need the **Owner** or **User Access Administrator** RBAC role on the Key Vault scope to assign the **Key Vault Crypto Officer** role. Contact your admin if needed.

#### [Azure portal](#tab/portal)

To assign the **Key Vault Crypto Officer** role to yourself by using the Azure portal, follow these steps:

1. Go to your Key Vault.
1. From the service menu, select **Access control (IAM)**.
1. Under **Grant access to this resource**, select **Add role assignment**.
1. Search for and select **Key Vault Crypto Officer**, and then select **Next**.
1. Under **Assign access to**, select **User, group, or service principal**.
1. Under **Members**, choose **+Select members**.
1. Search for and select your own account, then choose **Select**.
1. Select **Review + assign**, and then **Review + assign** again.

#### [PowerShell](#tab/powershell)

To assign the **Key Vault Crypto Officer** role to yourself by using Azure PowerShell, run the following cmdlet. Replace `<key-vault-name>` with your own value.

```azurepowershell
$keyVault = Get-AzKeyVault -VaultName <key-vault-name>
$currentUser = (Get-AzContext).Account.Id

New-AzRoleAssignment `
    -SignInName $currentUser `
    -RoleDefinitionName "Key Vault Crypto Officer" `
    -Scope $keyVault.ResourceId
```

#### [Azure CLI](#tab/cli)

To assign the **Key Vault Crypto Officer** role to yourself by using Azure CLI, run the following commands. Replace `<key-vault-name>` with your own value.

```azurecli
KV_RESOURCE_ID=$(az keyvault show --name <key-vault-name> --query id -o tsv)
CURRENT_USER=$(az ad signed-in-user show --query id -o tsv)

az role assignment create \
    --assignee-object-id $CURRENT_USER \
    --assignee-principal-type User \
    --role "Key Vault Crypto Officer" \
    --scope $KV_RESOURCE_ID
```

---

## Step 2: Add an encryption key

You need an RSA or RSA-HSM key of size 2048, 3072, or 4096. Create or import an RSA key in your Key Vault that Azure uses to encrypt your data. Before you create a key, make sure you have the **Key Vault Crypto Officer** role on the Key Vault.

### [Azure portal](#tab/portal)

To add an encryption key by using the Azure portal, follow these steps.

1. Go to your Key Vault in the Azure portal.
1. From the service menu, under **Objects**, select **Keys**.
1. Select **Generate/Import**.
1. Set **Key type** to **RSA** and **RSA key size** to **2048** (or 3072/4096).
1. Enter a name for the key and select **Create**.

### [PowerShell](#tab/powershell)

To add an encryption key by using Azure PowerShell, run the following cmdlet. Replace `<key-vault-name>` and `<key-name>` with your own values. For `-Size`, you can specify 2048, 3072, or 4096.

```azurepowershell
Add-AzKeyVaultKey `
    -VaultName <key-vault-name> `
    -Name <key-name> `
    -Destination Software `
    -KeyType RSA `
    -Size 2048
```

### [Azure CLI](#tab/cli)

To add an encryption key by using Azure CLI, run the following command. Replace `<key-vault-name>` and `<key-name>` with your own values. For `--size`, you can specify 2048, 3072, or 4096.

```azurecli
az keyvault key create \
    --vault-name <key-vault-name> \
    --name <key-name> \
    --kty RSA \
    --size 2048
```

---

## Step 3: Create a managed identity and assign permissions

The storage account needs a [managed identity](/entra/identity/managed-identities-azure-resources/overview) to authenticate to the Key Vault. By using a managed identity, the storage account can securely access the encryption key in your Key Vault without storing credentials.

Create a user-assigned or system-assigned managed identity and grant that identity the **Key Vault Crypto Service Encryption User** role on the Key Vault.

> [!IMPORTANT]
> If you plan to create a new storage account instead of using an existing storage account, you must create a user-assigned managed identity. You can't use a system-assigned identity during storage account creation because the identity doesn't exist until the storage account is created.

### Option A: Create a user-assigned managed identity

You can create a user-assigned managed identity by using the Azure portal, Azure PowerShell, or Azure CLI.

#### [Azure portal](#tab/portal)

To create a user-assigned managed identity by using the Azure portal, follow these steps.

1. Search for **Managed Identities** and select **Create**.
1. Choose a subscription, resource group, region, and name.
1. Select **Review + create**, and then select **Create**.

#### [PowerShell](#tab/powershell)

To create a user-assigned managed identity by using Azure PowerShell, run the following cmdlet. Replace `<resource-group>`, `<identity-name>`, and `<region>` with your own values.

```azurepowershell
New-AzUserAssignedIdentity `
    -ResourceGroupName <resource-group> `
    -Name <identity-name> `
    -Location <region>
```

#### [Azure CLI](#tab/cli)

To create a user-assigned managed identity by using Azure CLI, run the following command. Replace `<resource-group>`, `<identity-name>`, and `<region>` with your own values.

```azurecli
az identity create \
    --name <identity-name> \
    --resource-group <resource-group> \
    --location <region>
```

---

#### Assign the Key Vault Crypto Service Encryption User role

Assign the **Key Vault Crypto Service Encryption User** role to the user-assigned managed identity you created by using the Azure portal, PowerShell, or Azure CLI.

##### [Azure portal](#tab/portal)

To assign the **Key Vault Crypto Service Encryption User** role to the user-assigned managed identity by using the Azure portal, follow these steps:

1. Go to your Key Vault in the Azure portal.
1. From the service menu, select **Access control (IAM)**.
1. Under **Grant access to this resource**, select **Add role assignment**.
1. Search for and select **Key Vault Crypto Service Encryption User**, and then select **Next**.
1. Under **Assign access to**, select **Managed identity**.
1. Under **Members**, choose **+Select members**.
1. The **Select managed identities** window opens. Under **Managed identity**, select **User-assigned managed identity**.
1. Select the user-assigned managed identity that you created, and then choose **Select**.
1. Select **Review + assign**, and then **Review + assign** again.

##### [PowerShell](#tab/powershell)

To assign the **Key Vault Crypto Service Encryption User** role to the user-assigned managed identity by using Azure PowerShell, run the following cmdlet. Replace `<resource-group>`, `<identity-name>`, and `<key-vault-name>` with your own values.

```azurepowershell
$identity = Get-AzUserAssignedIdentity `
    -ResourceGroupName <resource-group> `
    -Name <identity-name>

$keyVault = Get-AzKeyVault -VaultName <key-vault-name>

New-AzRoleAssignment `
    -ObjectId $identity.PrincipalId `
    -RoleDefinitionName "Key Vault Crypto Service Encryption User" `
    -Scope $keyVault.ResourceId
```

##### [Azure CLI](#tab/cli)

To assign the **Key Vault Crypto Service Encryption User** role to the user-assigned managed identity by using Azure CLI, run the following commands. Replace `<resource-group>`, `<identity-name>`, and `<key-vault-name>` with your own values.

```azurecli
# Get the principal ID of the user-assigned managed identity
IDENTITY_PRINCIPAL_ID=$(az identity show \
    --name <identity-name> \
    --resource-group <resource-group> \
    --query principalId -o tsv)

# Get the key vault resource ID
KV_RESOURCE_ID=$(az keyvault show \
    --name <key-vault-name> \
    --query id -o tsv)

# Assign the Key Vault Crypto Service Encryption User role to the user-assigned managed identity
az role assignment create \
    --assignee-object-id $IDENTITY_PRINCIPAL_ID \
    --assignee-principal-type ServicePrincipal \
    --role "Key Vault Crypto Service Encryption User" \
    --scope $KV_RESOURCE_ID
```

---

You can now proceed to [Step 4: Configure customer-managed keys on the storage account](#step-4-configure-customer-managed-keys-on-the-storage-account).

### Option B: System-assigned managed identity (existing storage accounts only)

For existing storage accounts, you can enable a system-assigned managed identity for the storage account and assign it the **Key Vault Crypto Service Encryption User** role. You can do this by using the Azure portal, PowerShell, or Azure CLI.

#### [Azure portal](#tab/portal)

To enable a system-assigned managed identity on an existing storage account by using the Azure portal, follow these steps:

1. Go to your storage account in the Azure portal.
1. From the service menu, under **Security + networking**, select **Encryption**.
1. For **Encryption type**, select **Customer-managed keys**.
1. Under **Key vault and key**, select the Key Vault and key that you created in Step 2.
1. For **Identity type**, select **System assigned**.
1. Select **Save**.

Next, assign the **Key Vault Crypto Service Encryption User** role to the system-assigned managed identity for your storage account:

1. Go to your Key Vault.
1. From the service menu, select **Access control (IAM)**.
1. Under **Grant access to this resource**, select **Add role assignment**.
1. Search for and select **Key Vault Crypto Service Encryption User**, and then select **Next**.
1. Under **Assign access to**, select **Managed identity**.
1. Under **Members**, choose **+Select members**.
1. Search for and select the system-assigned managed identity for your storage account, and then choose **Select**.
1. Select **Review + assign**, and then **Review + assign** again.

#### [PowerShell](#tab/powershell)

To enable a system-assigned managed identity on an existing storage account and add the required role assignment by using Azure PowerShell, run the following cmdlets. Replace `<resource-group>`, `<storage-account-name>`, and `<key-vault-name>` with your own values.

```azurepowershell
# Enable system-assigned identity
Set-AzStorageAccount `
    -ResourceGroupName <resource-group> `
    -Name <storage-account-name> `
    -AssignIdentity

$storageAccount = Get-AzStorageAccount `
    -ResourceGroupName <resource-group> `
    -Name <storage-account-name>

$keyVault = Get-AzKeyVault -VaultName <key-vault-name>

New-AzRoleAssignment `
    -ObjectId $storageAccount.Identity.PrincipalId `
    -RoleDefinitionName "Key Vault Crypto Service Encryption User" `
    -Scope $keyVault.ResourceId
```

#### [Azure CLI](#tab/cli)

To enable a system-assigned managed identity on an existing storage account and add the required role assignment by using Azure CLI, run the following commands. Replace `<resource-group>`, `<storage-account-name>`, and `<key-vault-name>` with your own values.

```azurecli
# Enable system-assigned identity on the storage account
az storage account update \
    --name <storage-account-name> \
    --resource-group <resource-group> \
    --assign-identity

# Get the principal ID
SA_PRINCIPAL_ID=$(az storage account show \
    --name <storage-account-name> \
    --resource-group <resource-group> \
    --query identity.principalId -o tsv)

# Assign the role on the key vault
KV_RESOURCE_ID=$(az keyvault show --name <key-vault-name> --query id -o tsv)

az role assignment create \
    --assignee-object-id $SA_PRINCIPAL_ID \
    --assignee-principal-type ServicePrincipal \
    --role "Key Vault Crypto Service Encryption User" \
    --scope $KV_RESOURCE_ID
```

---

## Step 4: Configure customer-managed keys on the storage account

With the Key Vault, key, and managed identity in place, you can enable customer-managed keys on the storage account.

Follow these steps to configure the storage account to use your key from Key Vault for encryption. Choose between automatic key version updates (recommended) or manual key version management.

> [!IMPORTANT]
> For storage accounts that are associated with a [network security perimeter](files-network-security-perimeter.md), the Key Vault should ideally be in the same network security perimeter. If it isn't, then you must configure the network security perimeter profile of the Key Vault to allow the storage account to communicate with it.

### Configure customer-managed keys for an existing storage account

You can configure customer-managed keys on an existing storage account by using the Azure portal, PowerShell, or Azure CLI.

#### [Azure portal](#tab/portal)

To configure customer-managed keys on an existing storage account by using the Azure portal, follow these steps:

1. Go to your storage account.
1. From the service menu, under **Security + networking**, select **Encryption**.
1. For **Encryption type**, select **Customer-Managed Keys**. If the storage account is already configured for CMK, you need to select **Change key**.
1. For **Encryption key**, select **Select from key vault**.
1. Select **Select a Key Vault and key**, and then choose your Key Vault and key. To enable *automatic key version rotation*, don't specify a key version when selecting the key. Azure checks for new key versions daily and automatically updates.
1. For **Identity type**, choose:
   - **System-assigned** - uses the storage account's system identity, which is created automatically.
   - **User-assigned** - uses your previously created user-assigned managed identity.
1. If you selected **User-assigned**, you must search for and select the user-assigned managed identity and then select **Add**.
1. Select **Save**.

:::image type="content" source="media/customer-managed-keys/encryption-key-selection.png" alt-text="Screenshot showing the encryption selection and key selection for configuring customer managed keys.":::

#### [PowerShell](#tab/powershell)

To configure customer-managed keys on an existing storage account by using Azure PowerShell with automatic key version updating (recommended), run the following cmdlet. Replace `<resource-group>`, `<identity-name>`, `<storage-account-name>`, `<key-vault-name>`, and `<key-name>` with your own values.

The following cmdlet uses your previously created user-assigned managed identity. If you want to use the storage account's system identity instead, set `IdentityType` to `SystemAssigned` and omit the `$identity` variable, `UserAssignedIdentityId`, and `KeyVaultUserAssignedIdentityId`.

```azurepowershell
$identity = Get-AzUserAssignedIdentity `
    -ResourceGroupName <resource-group> `
    -Name <identity-name>

$keyVault = Get-AzKeyVault -VaultName <key-vault-name>
$key = Get-AzKeyVaultKey -VaultName <key-vault-name> -Name <key-name>

Set-AzStorageAccount `
    -ResourceGroupName <resource-group> `
    -Name <storage-account-name> `
    -IdentityType UserAssigned `
    -UserAssignedIdentityId $identity.Id `
    -KeyVaultUri $keyVault.VaultUri `
    -KeyName $key.Name `
    -KeyVaultEncryption `
    -KeyVaultUserAssignedIdentityId $identity.Id
```

To use manual key version updating, run the following cmdlet. Replace `<resource-group>`, `<identity-name>`, `<storage-account-name>`, `<key-vault-name>`, and `<key-name>` with your own values.

The following cmdlet uses your previously created user-assigned managed identity. If you want to use the storage account's system identity instead, set `IdentityType` to `SystemAssigned` and omit the `$identity` variable, `UserAssignedIdentityId`, and `KeyVaultUserAssignedIdentityId`.

```azurepowershell
$identity = Get-AzUserAssignedIdentity `
    -ResourceGroupName <resource-group> `
    -Name <identity-name>

$keyVault = Get-AzKeyVault -VaultName <key-vault-name>
$key = Get-AzKeyVaultKey -VaultName <key-vault-name> -Name <key-name>

Set-AzStorageAccount `
    -ResourceGroupName <resource-group> `
    -Name <storage-account-name> `
    -IdentityType SystemAssignedUserAssigned `
    -UserAssignedIdentityId $identity.Id `
    -KeyVaultUri $keyVault.VaultUri `
    -KeyName $key.Name `
    -KeyVersion $key.Version `
    -KeyVaultEncryption `
    -KeyVaultUserAssignedIdentityId $identity.Id
```

#### [Azure CLI](#tab/cli)

To configure customer-managed keys on an existing storage account by using Azure CLI with automatic key version updating (recommended), run the following commands. Replace `<resource-group>`, `<identity-name>`, `<storage-account-name>`, `<key-vault-uri>`, and `<key-name>` with your own values.

The following command uses your previously created user-assigned managed identity. If you want to use the storage account's system identity instead, set `--identity-type` to `SystemAssigned` and omit the `IDENTITY_RESOURCE_ID` variable, `--user-identity-id`, and `--key-vault-user-identity-id`.

```azurecli
# Using user-assigned managed identity
IDENTITY_RESOURCE_ID=$(az identity show \
    --name <identity-name> \
    --resource-group <resource-group> \
    --query id -o tsv)

az storage account update \
    --name <storage-account-name> \
    --resource-group <resource-group> \
    --encryption-key-vault <key-vault-uri> \
    --encryption-key-name <key-name> \
    --encryption-key-source Microsoft.Keyvault \
    --key-vault-user-identity-id $IDENTITY_RESOURCE_ID \
    --identity-type UserAssigned \
    --user-identity-id $IDENTITY_RESOURCE_ID
```

To use manual key version updating, run the following command. Replace `<resource-group>`, `<storage-account-name>`, `<key-vault-uri>`, `<key-name>`, and `<key-version>` with your own values.

The following command uses your previously created user-assigned managed identity. If you want to use the storage account's system identity instead, set `--identity-type` to `SystemAssigned` and omit the `IDENTITY_RESOURCE_ID` variable, `--user-identity-id`, and `--key-vault-user-identity-id`.

```azurecli
# Using user-assigned managed identity
IDENTITY_RESOURCE_ID=$(az identity show \
    --name <identity-name> \
    --resource-group <resource-group> \
    --query id -o tsv)

az storage account update \
    --name <storage-account-name> \
    --resource-group <resource-group> \
    --encryption-key-vault <key-vault-uri> \
    --encryption-key-name <key-name> \
    --encryption-key-version <key-version> \
    --encryption-key-source Microsoft.Keyvault \
    --key-vault-user-identity-id $IDENTITY_RESOURCE_ID \
    --identity-type SystemAssigned,UserAssigned \
    --user-identity-id $IDENTITY_RESOURCE_ID
```

---

### Configure customer-managed keys for a new storage account

You can configure customer-managed keys for a new storage account by using the Azure portal, Azure PowerShell, or Azure CLI.

#### [Azure portal](#tab/portal)

To configure customer-managed keys for a new storage account by using the Azure portal, follow these steps:

1. On the **Encryption** tab during storage account creation, select **Customer-managed keys (CMK)** for **Encryption type**.
1. Choose **Select a Key Vault and key**, then pick your vault and key.
1. Select your pre-created **user-assigned managed identity**. New storage accounts can't use a system-assigned managed identity.
1. Complete the remaining tabs and select **Review + create**.

#### [PowerShell](#tab/powershell)

To create a new storage account with customer-managed keys by using Azure PowerShell, run the following cmdlet. Replace `<resource-group>`, `<identity-name>`, `<storage-account-name>`, `<key-vault-name>`, `<key-name>`, and `<region>` with your own values. You can specify different values for `-Kind` and `-SkuName` if desired.

```azurepowershell
$identity = Get-AzUserAssignedIdentity `
    -ResourceGroupName <resource-group> `
    -Name <identity-name>

$keyVault = Get-AzKeyVault -VaultName <key-vault-name>
$key = Get-AzKeyVaultKey -VaultName <key-vault-name> -Name <key-name>

New-AzStorageAccount `
    -ResourceGroupName <resource-group> `
    -Name <storage-account-name> `
    -Location <region> `
    -SkuName Standard_LRS `
    -Kind StorageV2 `
    -IdentityType UserAssigned `
    -UserAssignedIdentityId $identity.Id `
    -KeyVaultUri $keyVault.VaultUri `
    -KeyName $key.Name `
    -KeyVaultEncryption `
    -KeyVaultUserAssignedIdentityId $identity.Id
```

#### [Azure CLI](#tab/cli)

To create a new storage account with customer-managed keys by using Azure CLI, run the following commands. Replace `<resource-group>`, `<identity-name>`, `<storage-account-name>`, `<key-vault-uri>`, `<key-name>`, and `<region>` with your own values. You can specify different values for `--kind` and `--sku` if desired.

```azurecli
IDENTITY_RESOURCE_ID=$(az identity show \
    --name <identity-name> \
    --resource-group <resource-group> \
    --query id -o tsv)

az storage account create \
    --name <storage-account-name> \
    --resource-group <resource-group> \
    --location <region> \
    --sku Standard_LRS \
    --kind StorageV2 \
    --identity-type UserAssigned \
    --user-identity-id $IDENTITY_RESOURCE_ID \
    --encryption-key-vault <key-vault-uri> \
    --encryption-key-name <key-name> \
    --encryption-key-source Microsoft.Keyvault \
    --key-vault-user-identity-id $IDENTITY_RESOURCE_ID
```

---

## Step 5: Verify the configuration

After you enable customer-managed keys, confirm that encryption is properly configured on your storage account. You can do this by using the Azure portal, Azure PowerShell, or Azure CLI.

### [Azure portal](#tab/portal)

To verify the configuration by using the Azure portal, follow these steps:

1. Go to your storage account in the Azure portal.
1. From the service menu, under **Security + networking**, select **Encryption**.
1. Confirm that **Encryption type** shows **Customer-Managed Keys**.
1. Verify that the information under **Key selection** is correct.

### [PowerShell](#tab/powershell)

To verify the configuration by using Azure PowerShell, run the following cmdlet. Replace `<resource-group>` and `<storage-account-name>` with your own values. Confirm that `keySource` is `Microsoft.Keyvault` and the `keyvaultproperties` section shows your Key Vault URI and key name.

```azurepowershell
$account = Get-AzStorageAccount `
    -ResourceGroupName <resource-group> `
    -Name <storage-account-name>

$account.Encryption
```

### [Azure CLI](#tab/cli)

To verify the configuration by using Azure CLI, run the following command. Replace `<resource-group>` and `<storage-account-name>` with your own values. Confirm that `keySource` is `Microsoft.Keyvault` and the `keyvaultproperties` section shows your Key Vault URI and key name.

```azurecli
az storage account show \
    --name <storage-account-name> \
    --resource-group <resource-group> \
    --query encryption
```

---

## Key rotation

Regularly rotating your encryption key limits the exposure if a key is ever compromised. For security best practices, rotate your encryption key at least once every two years.

- **Automated key rotation (recommended):** If you configured customer-managed keys without specifying a key version, Azure checks for new key versions daily. If you create a new version of the key in Key Vault, Azure picks it up within 24 hours. You can also configure [automatic key rotation in Azure Key Vault](/azure/key-vault/keys/how-to-configure-key-rotation) to generate new key versions on a schedule.
- **Manual key rotation:** If you specified a key version when configuring customer-managed keys, you must manually update the storage account configuration to point to the new key version.

> [!IMPORTANT]
> Azure checks the Key Vault for a new key version only once daily. After rotating a key, wait 24 hours before disabling the previous key version.

### Manually rotate the encryption key

To manually rotate the encryption key, use the Azure portal to update the storage account encryption configuration to point to the new key version. Follow these steps:

1. Go to your storage account in the Azure portal.
1. From the service menu, under **Security + networking**, select **Encryption**.
1. In the **Key selection** section, select **Change key**.
1. Under **Key vault and key**, choose **Select a key vault and key**.
1. Select your Key Vault and key, and then select the new key version.
1. Select **Save**.

## Revoke access to file share data by disabling the key

You can immediately block access to encrypted file share data by disabling or deleting the customer-managed key. While the key is disabled, all Azure Files data plane operations fail with HTTP 403 (Forbidden), including:

- List directories and files
- Create/get/set directory or file
- Get/set file metadata
- Put range, copy file, rename file

To revoke access to your file share data, disable the key in Key Vault by using the Azure portal, Azure PowerShell, or Azure CLI. Re-enable the key to restore access.

### [Azure portal](#tab/portal)

To disable the key by using the Azure portal, follow these steps:

1. Go to your Key Vault in the Azure portal.
1. From the service menu, under **Objects**, select **Keys**.
1. Right-click the key and select **Disable**.

### [PowerShell](#tab/powershell)

To disable the key by using Azure PowerShell, run the following cmdlet. Replace `<key-vault-name>` and `<key-name>` with your own values.

```azurepowershell
Update-AzKeyVaultKey `
    -VaultName <key-vault-name> `
    -Name <key-name> `
    -Enable $false
```

### [Azure CLI](#tab/cli)

To disable the key by using Azure CLI, run the following command. Replace `<key-vault-name>` and `<key-name>` with your own values.

```azurecli
az keyvault key set-attributes \
    --vault-name <key-vault-name> \
    --name <key-name> \
    --enabled false
```

---

## Switch back to Microsoft-managed keys

If you no longer need customer-managed keys, you can switch the storage account back to using Microsoft-managed keys for encryption by using the Azure portal, Azure PowerShell, or Azure CLI.

### [Azure portal](#tab/portal)

To switch back to Microsoft-managed keys by using the Azure portal, follow these steps:

1. Go to your storage account in the Azure portal.
1. From the service menu, under **Security + networking**, select **Encryption**.
1. Change **Encryption type** to **Microsoft-Managed Keys**.
1. Select **Save**.

### [PowerShell](#tab/powershell)

To switch back to Microsoft-managed keys by using Azure PowerShell, run the following cmdlet. Replace `<resource-group>` and `<storage-account-name>` with your own values.

```azurepowershell
Set-AzStorageAccount `
    -ResourceGroupName <resource-group> `
    -Name <storage-account-name> `
    -StorageEncryption
```

### [Azure CLI](#tab/cli)

To switch back to Microsoft-managed keys by using Azure CLI, run the following command. Replace `<resource-group>` and `<storage-account-name>` with your own values.

```azurecli
az storage account update \
    --name <storage-account-name> \
    --resource-group <resource-group> \
    --encryption-key-source Microsoft.Storage
```

---

## Related content

For more information about encryption and key management, see the following articles.

- [Azure Storage encryption for data at rest](/azure/storage/common/storage-service-encryption)
- [Customer-managed keys for Azure Storage encryption (overview)](/azure/storage/common/customer-managed-keys-overview)
- [Configure cross-tenant customer-managed keys](/azure/storage/common/customer-managed-keys-configure-cross-tenant-existing-account)
- [Encryption for Azure Files](/azure/storage/files/storage-files-planning#encryption-for-azure-files)
- [What is Azure Key Vault?](/azure/key-vault/general/overview)
