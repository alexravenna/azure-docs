---
#customer intent: As a developer, I want to understand my secrets options for Azure Functions on Azure Container Apps so I can choose the right approach.
title: Managing secrets for Azure Functions on Azure Container Apps
description: Understand the two categories of secrets for Azure Functions on Azure Container Apps and choose the right implementation path.
author: deepganguly
ms.author: deepganguly
ms.reviewer: cshoe
ms.service: azure-container-apps
ms.topic: overview
ms.date: 04/27/2026
---

# Managing secrets for Azure Functions on Azure Container Apps

## Overview

When you run Azure Functions on Azure Container Apps, you work with two distinct categories of secrets:

| Category | What it is | Who consumes it |
|----------|-----------|----------------|
| **App-level secrets** | Configuration values your function code reads at runtime - database connection strings, API keys, trigger/binding connection strings. | Your code and the Functions runtime bindings. |
| **Functions access keys** | Authentication tokens ([access keys](/azure/azure-functions/function-keys-how-to#understand-keys)) that secure HTTP-triggered function endpoints - master, host, function, and system keys. | Callers of your HTTP functions (other services, webhooks, developers). |

The key distinction is **direction**:

- **App-level secrets** are **outbound** - your function uses them to authenticate when connecting to other services (databases, APIs, queues).
- **Functions access keys** are **inbound** - callers pass them to authenticate when invoking your HTTP endpoints.

| | App-level secrets | Functions access keys |
|---|---|---|
| **Direction** | Outbound - your function calls other services | Inbound - callers invoke your function |
| **Who holds the secret** | Your function app | The caller (webhook provider, another service, a developer) |
| **What it protects** | What your function can connect to | Who can invoke your HTTP endpoint |
| **Validated by** | The target service | The Functions runtime |

Each category has its own storage options, trade-offs, and configuration steps. Choose the guide that matches what you need:

## App-level secrets

Store configuration values that your function code and bindings consume. Two options:

| Option | Best for | Rotation | Audit | Guide |
|--------|---------|----------|-------|-------|
| **Container Apps secrets** | Dev/test, simple single-app workloads | Manual | Activity logs only | [Store app-level secrets](functions-secrets-app-level.md#use-container-apps-secrets) |
| **Key Vault references** | Production, multi-app, compliance | Automatic (versionless URI) | Full Key Vault diagnostics | [Store app-level secrets](functions-secrets-app-level.md#use-key-vault-references) |

> [!TIP]
> Start with Container Apps secrets for simplicity. Move to Key Vault references when you need centralized management, automatic rotation, or compliance-grade auditing.

## Functions access keys

Lightweight shared-secret auth for HTTP endpoints. Use [access keys](/azure/azure-functions/function-keys-how-to#understand-keys) when callers can't use Microsoft Entra ID tokens - for example, third-party webhooks, service-to-service calls, or during development.

You choose a **storage backend** via the `AzureWebJobsSecretStorageType` environment variable:

> [!IMPORTANT]
> **For production, choose backends in this order:** Container Apps secret store (`containerapp`) > Azure Key Vault (`keyvault`) > Azure Blob Storage (`blob`). Start with the Container Apps secret store unless you have a specific reason to use the others.

| Backend | Setting value | Best for | Production rank | Guide |
|---------|--------------|---------|-----------------|-------|
| **Container Apps secret store** | `containerapp` | Most Container Apps workloads - no external dependencies, you provision keys as Container Apps secrets | **1st - Recommended** | [Configure host keys](functions-secrets-host-keys.md#configure-the-container-apps-secret-store) |
| **Azure Key Vault** | `keyvault` | Centralized governance, compliance auditing | **2nd** - when you need cross-app governance or compliance audit | [Configure host keys](functions-secrets-host-keys.md#configure-key-vault) |
| **Azure Blob Storage** | `blob` | Legacy or existing `AzureWebJobsStorage` dependency | **3rd** - only with existing storage dependency | [Configure host keys](functions-secrets-host-keys.md#configure-blob-storage) |

> [!WARNING]
> The **default** backend is `files` (local file system). On Azure Container Apps, the file system is **ephemeral** - host keys are lost on every restart, scale-to-zero, or revision deploy. **Always** configure one of the three production backends above.

## Prerequisites

Both guides share these prerequisites:

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/).
- [Azure CLI](/cli/azure/install-azure-cli) version 2.40.0 or higher.
- An existing [Azure Functions app in Container Apps](functions-usage.md) or permissions to create one.

## Related content

- [Store app-level secrets for Functions on Container Apps](functions-secrets-app-level.md)
- [Configure Functions host key storage on Container Apps](functions-secrets-host-keys.md)
- [Manage secrets in Azure Container Apps](manage-secrets.md)
- [Azure Functions on Azure Container Apps overview](functions-overview.md)
- [Managed identities in Container Apps](managed-identity.md)
