---
title: 'Quickstart: Deploy an Express container app using the Azure CLI'
description: Learn how to deploy your first Azure Container Apps express app using the Azure CLI, including creating an express environment and deploying a container image.
ms.service: azure-container-apps
author: craigshoemaker
ms.author: cshoe
ms.service: azure-container-apps
ms.topic: quickstart
ms.date: 05/05/2026
# customer intent: As a developer, I want to deploy a container app to Azure Container Apps express using the Azure CLI so that I can get my web app running in the cloud as quickly as possible.
---

# Quickstart: Deploy an Express container app by using the Azure CLI (private preview)

In this quickstart, you use the Azure CLI to create an Azure Container Apps express environment and deploy your first express container app. Azure Container Apps express is a developer-first platform that gets your containerized web app running in the cloud with minimal configuration.

> [!IMPORTANT]
> Azure Container Apps express is currently in private preview. Access is limited to Microsoft Entra ID accounts. Personal Microsoft accounts aren't supported.

## Prerequisites

- An Azure account with an active subscription. If you don't have one, [create an account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- A **Microsoft Entra ID account**. Only Entra ID accounts can sign in to express. Non-Entra ID accounts, such as personal Microsoft accounts, aren't supported.
- The [Azure CLI](/cli/azure/install-azure-cli) installed.

## Update the Container Apps extension

Before you begin, upgrade the Azure Container Apps CLI extension to the required version.

```azurecli
az extension update --name containerapp
```

> [!NOTE]
> You need version **1.3.0b4** or later of the `containerapp` extension.

## Create an express environment

Create a resource group and an express environment. Replace `<ENVIRONMENT_NAME>` and `<RESOURCE_GROUP>` with your own values.

```azurecli
az containerapp env create \
  --environment-mode express \
  --name <ENVIRONMENT_NAME> \
  --resource-group <RESOURCE_GROUP> \
  --logs-destination none
```

> [!NOTE]
> During the private preview, express is available only in the **West Central US** region. Ensure your resource group targets that region.

## Deploy a container app

Deploy a container image to the express environment.

```azurecli
az containerapp up \
  --image docker.io/nginx \
  --name <APP_NAME> \
  --resource-group <RESOURCE_GROUP>
```

After the command completes, the CLI outputs the URL for your running app.

## Provide feedback

If you encounter issues or have feedback during the private preview, file an issue on the [Azure Container Apps GitHub repository](https://github.com/microsoft/azure-container-apps/). Start your issue title with **[EPP]** to identify it as an Express Private Preview issue.

For example: `[EPP] Deployment fails when using custom environment variables`

## Related content

- [Azure Container Apps express overview](express-overview.md)
- [Azure Container Apps environments](environment.md)
- [Azure Container Apps jobs](jobs.md)
