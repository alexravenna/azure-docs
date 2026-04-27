---
#customer intent: As a developer, I want to configure durable storage for Functions host keys on Azure Container Apps so my HTTP-triggered functions stay secured across restarts.
title: Configure Functions host key storage on Azure Container Apps
description: Choose and configure a storage backend for Functions host keys - Container Apps secret store, Key Vault, or Blob Storage - for Azure Functions on Azure Container Apps.
author: deepganguly
ms.author: deepganguly
ms.reviewer: cshoe
ms.service: azure-container-apps
ms.topic: how-to
ms.date: 04/27/2026
---

# Configure Functions host key storage on Azure Container Apps

## What are Functions host keys?

Functions host keys are authentication tokens that the Functions runtime uses to secure HTTP-triggered endpoints. When a caller invokes an HTTP function, it passes a key as a `?code=` query parameter or an `x-functions-key` header. The runtime validates the key against its secret store and authorizes (or rejects) the request.

Host keys are **not** the same as [app-level secrets](functions-secrets-app-level.md), which store credentials your code reads at runtime (database passwords, API keys, etc.). Host keys protect **who can call your functions**, while app-level secrets protect **what your functions connect to**.

## When to use host keys

| Scenario | Why host keys fit |
|----------|------------------|
| **Third-party webhooks** | Providers like GitHub, Stripe, or Twilio call your function via a URL + secret. Host keys slot directly into the `?code=` pattern they expect. |
| **Service-to-service calls in a trusted network** | Backend service A calls Function B over HTTP. Passing a shared key is simpler than setting up Microsoft Entra app registrations for an internal-only call. |
| **Event Grid subscriptions** | Event Grid validates and calls your function endpoint using a **system key** - the platform manages this automatically. |
| **Dev/test quick auth gate** | During development you need *some* authentication without wiring up full OAuth/OIDC. Host keys provide a non-zero auth bar with zero identity config. |
| **Migration compatibility** | Existing Azure Functions apps already use host keys. When migrating to Container Apps, you need the same key-based auth to avoid breaking callers. |

> [!NOTE]
> For user-facing APIs, zero-trust workloads, or scenarios requiring per-user authorization, use Microsoft Entra ID / OAuth 2.0 instead of host keys. Host keys are shared secrets with no identity-level audit trail.

This article explains the key types the Functions runtime manages, the storage backends available on Azure Container Apps, and how to configure each one.

## Prerequisites

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/).
- [Azure CLI](/cli/azure/install-azure-cli) version 2.40.0 or higher.
- An existing [Azure Functions app in Container Apps](functions-usage.md) or permissions to create one.

## Host key types

The Functions runtime manages four types of keys:

| Key type | Scope | Purpose |
|----------|-------|---------|
| **Master key** (`_master`) | Entire function app | Admin-level access to all functions and the `/admin/*` management endpoints. Can't be revoked, only rotated. |
| **Host keys** (`default` + custom) | Entire function app | Authorize calls to any HTTP-triggered function in the app. |
| **Function keys** (`default` + custom) | Single function | Authorize calls to one specific function. More granular than host keys. |
| **System keys** | Extension endpoints | Used by platform extensions (for example, Event Grid webhook subscriptions, Durable Functions). Managed automatically; you rarely interact with these directly. |

## Host key secret name patterns

The Functions runtime stores keys in the configured backend using these naming conventions. These are the names you see in Blob Storage containers, Key Vault secrets, or the Container Apps secret store.

| Key type | Secret name pattern | Example |
|----------|--------------------|---------| 
| Master key | `host--masterKey--master` | `host--masterKey--master` |
| Host key (default) | `host--hostKey--default` | `host--hostKey--default` |
| Host key (custom) | `host--hostKey--<name>` | `host--hostKey--MyCustomKey` |
| Function key (default) | `host--functionKey--default` | `host--functionKey--default` |
| Function key (custom) | `host--functionKey--<name>` | `host--functionKey--MyApiClient` |
| System key | `host--systemKey--<extension>` | `host--systemKey--eventgrid_extension` |

> [!TIP]
> When troubleshooting, search for these patterns in your backend store to verify that the runtime created keys successfully.

## Choose a storage backend

You control where the runtime persists host keys by setting the `AzureWebJobsSecretStorageType` environment variable. Azure Container Apps supports three production-grade backends.

> [!IMPORTANT]
> **For production, choose backends in this order:** Container Apps secret store (`containerapp`) > Azure Key Vault (`keyvault`) > Azure Blob Storage (`blob`). The ACA secret store has no external dependencies and is the simplest to operate. Use Key Vault only when you need centralized cross-app governance or compliance-grade audit logs. Use Blob Storage only when you have an existing storage account dependency.

| Backend | Setting value | Auto-generates keys | External dependency | Best for | Production rank |
|---------|--------------|--------------------|--------------------|----------|----------------|
| **Container Apps secret store** | `containerapp` | Yes | None | Most workloads on ACA - simplest, no external resources | **1st - Recommended** |
| **Azure Key Vault** | `keyvault` | No - trigger creation manually | Key Vault instance | Centralized governance, compliance, enterprise auditing | **2nd** |
| **Azure Blob Storage** | `blob` | Yes | Storage account | Legacy or when you already share an `AzureWebJobsStorage` account | **3rd** |

> [!WARNING]
> The **default** backend is `files` (local file system). On Azure Container Apps, the file system is **ephemeral** - host keys stored with `files` are lost every time the app scales to zero, restarts, or a new revision deploys. **Always** set `AzureWebJobsSecretStorageType` to one of the three production backends above.

## Configure the Container Apps secret store

The Container Apps secret store is the recommended backend. Keys stay within the ACA platform boundary, require no external storage or Key Vault, and integrate with the same `az containerapp` CLI commands you use for other app secrets.

1. Set the storage type:

    ```azurecli
    az containerapp update \
      --resource-group "<RESOURCE_GROUP>" \
      --name "<FUNCTIONS_APP_NAME>" \
      --set-env-vars "AzureWebJobsSecretStorageType=containerapp"
    ```

1. The Functions runtime auto-generates keys on the next cold start. Verify by listing host keys:

    ```azurecli
    az containerapp function keys list \
      --resource-group "<RESOURCE_GROUP>" \
      --name "<FUNCTIONS_APP_NAME>" \
      --key-type hostKey
    ```

## Configure Blob Storage

The Blob Storage backend lets the runtime auto-generate and manage host keys. Use this option when you already have a storage account for `AzureWebJobsStorage` and don't need centralized cross-app governance.

1. Enable managed identity on your container app (if not already enabled):

    ```azurecli
    az containerapp identity assign \
      --resource-group "<RESOURCE_GROUP>" \
      --name "<FUNCTIONS_APP_NAME>" \
      --system-assigned
    ```

1. Grant the **Storage Blob Data Contributor** role on the storage account to the managed identity:

    ```azurecli
    PRINCIPAL_ID=$(az containerapp show \
      --resource-group "<RESOURCE_GROUP>" \
      --name "<FUNCTIONS_APP_NAME>" \
      --query identity.principalId \
      --output tsv)

    STORAGE_ID=$(az storage account show \
      --name "<STORAGE_ACCOUNT_NAME>" \
      --resource-group "<RESOURCE_GROUP>" \
      --query id \
      --output tsv)

    az role assignment create \
      --role "Storage Blob Data Contributor" \
      --assignee "$PRINCIPAL_ID" \
      --scope "$STORAGE_ID"
    ```

1. Set the storage type:

    ```azurecli
    az containerapp update \
      --resource-group "<RESOURCE_GROUP>" \
      --name "<FUNCTIONS_APP_NAME>" \
      --set-env-vars "AzureWebJobsSecretStorageType=blob"
    ```

1. The runtime auto-generates keys on the next cold start. Verify:

    ```azurecli
    az containerapp function keys list \
      --resource-group "<RESOURCE_GROUP>" \
      --name "<FUNCTIONS_APP_NAME>" \
      --key-type hostKey
    ```

## Configure Key Vault

The Key Vault backend stores host keys as Key Vault secrets, providing enterprise-grade auditing and access control.

1. Create a Key Vault (if you don't have one):

    ```azurecli
    az keyvault create \
      --name "<KEYVAULT_NAME>" \
      --resource-group "<RESOURCE_GROUP>" \
      --location "<LOCATION>"
    ```

1. Enable managed identity on your container app (if not already enabled):

    ```azurecli
    az containerapp identity assign \
      --resource-group "<RESOURCE_GROUP>" \
      --name "<FUNCTIONS_APP_NAME>" \
      --system-assigned
    ```

1. Grant the **Key Vault Secrets Officer** role to the managed identity. The runtime needs read and write access to create and manage keys:

    ```azurecli
    PRINCIPAL_ID=$(az containerapp show \
      --resource-group "<RESOURCE_GROUP>" \
      --name "<FUNCTIONS_APP_NAME>" \
      --query identity.principalId \
      --output tsv)

    KEYVAULT_ID=$(az keyvault show \
      --name "<KEYVAULT_NAME>" \
      --query id \
      --output tsv)

    az role assignment create \
      --role "Key Vault Secrets Officer" \
      --assignee "$PRINCIPAL_ID" \
      --scope "$KEYVAULT_ID"
    ```

1. Set the storage type and Key Vault URI:

    For system-assigned identity:

    ```azurecli
    az containerapp update \
      --resource-group "<RESOURCE_GROUP>" \
      --name "<FUNCTIONS_APP_NAME>" \
      --set-env-vars \
        "AzureWebJobsSecretStorageType=keyvault" \
        "AzureWebJobsSecretStorageKeyVaultUri=https://<KEYVAULT_NAME>.vault.azure.net"
    ```

    For user-assigned identity, also set the client ID:

    ```azurecli
    az containerapp update \
      --resource-group "<RESOURCE_GROUP>" \
      --name "<FUNCTIONS_APP_NAME>" \
      --set-env-vars \
        "AzureWebJobsSecretStorageType=keyvault" \
        "AzureWebJobsSecretStorageKeyVaultUri=https://<KEYVAULT_NAME>.vault.azure.net" \
        "AzureWebJobsSecretStorageKeyVaultClientId=<USER_ASSIGNED_IDENTITY_CLIENT_ID>"
    ```

1. Trigger key creation by listing keys:

    ```azurecli
    az containerapp function keys list \
      --resource-group "<RESOURCE_GROUP>" \
      --name "<FUNCTIONS_APP_NAME>" \
      --key-type hostKey
    ```

## Manage host keys

Regardless of backend, use the following commands to list, create, and delete host keys:

```azurecli
# List all host keys
az containerapp function keys list \
  --resource-group "<RESOURCE_GROUP>" \
  --name "<FUNCTIONS_APP_NAME>" \
  --key-type hostKey

# List the master key
az containerapp function keys list \
  --resource-group "<RESOURCE_GROUP>" \
  --name "<FUNCTIONS_APP_NAME>" \
  --key-type masterKey

# Create or overwrite a custom host key
az containerapp function keys set \
  --resource-group "<RESOURCE_GROUP>" \
  --name "<FUNCTIONS_APP_NAME>" \
  --key-name "MyCustomKey" \
  --key-value "<YOUR_KEY_VALUE>" \
  --key-type hostKey

# Show a specific key
az containerapp function keys show \
  --resource-group "<RESOURCE_GROUP>" \
  --name "<FUNCTIONS_APP_NAME>" \
  --key-name "<KEY_NAME>" \
  --key-type hostKey

# Delete a host key
az containerapp function keys delete \
  --resource-group "<RESOURCE_GROUP>" \
  --name "<FUNCTIONS_APP_NAME>" \
  --key-name "MyCustomKey" \
  --key-type hostKey
```

## Call a function with a host key

Pass the key as a query parameter or request header:

```bash
# Query parameter
curl "https://<FUNCTIONS_APP_URL>/api/<FUNCTION_NAME>?code=<HOST_KEY>"

# Header
curl "https://<FUNCTIONS_APP_URL>/api/<FUNCTION_NAME>" \
  -H "x-functions-key: <HOST_KEY>"
```

## Related content

- [Functions secrets overview](functions-secrets-tutorial.md)
- [Store app-level secrets](functions-secrets-app-level.md)
- [Manage secrets in Azure Container Apps](manage-secrets.md)
- [Manage Functions keys](functions-manage.md)
- [Managed identities in Container Apps](managed-identity.md)
- [Azure Functions on Azure Container Apps overview](functions-overview.md)
