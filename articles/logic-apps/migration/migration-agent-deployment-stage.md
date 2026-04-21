---
title: "Stage 5 - Deployment: Deploy to Azure"
titleSuffix: Logic Apps Migration Agent - Azure
description: Learn how the Deployment stage in the Logic Apps Migration Agent deploys your migrated Logic Apps Standard workflows to Azure.
author: haroldcampos
ms.author: hcampos
ms.reviewer: estfan
ms.topic: overview
ms.date: 04/06/2026
---

# Stage 5 - Deployment: Deploy to Azure

The Deployment stage is the final step of the Logic Apps Migration Agent workflow. In this stage, you deploy your validated Logic Apps Standard workflows and supporting resources to Azure.

## What happens during Deployment

The extension uses the Azure CLI to provision infrastructure and deploy your Logic Apps artifacts:

1. **Resource provisioning**: Creates the required Azure resources, including the Logic Apps Standard resource (Workflow Service Plan), storage account, and Application Insights instance.

1. **Artifact deployment**: Deploys the generated `workflow.json`, `connections.json`, `host.json`, and any .NET local functions to the provisioned Logic Apps resource.

1. **Connection authorization**: Configures managed connector connections and prompts for any authorization steps required by specific connectors.

## Prerequisites for Deployment

Before you deploy, make sure you have:

| Requirement | Purpose |
|---|---|
| Azure CLI | Provisions and deploys Azure resources |
| Azure subscription | Target subscription for deployment |
| Contributor access | Role-based access to create resources in the target resource group |

## Configure deployment settings

Before deployment, configure the extension settings at **Settings** > **Extensions** > **Logic Apps Migration Agent**:

| Setting | Description | Default |
|---|---|---|
| `logicAppsMigrationAssistant.azure.subscriptionId` | Your Azure subscription ID | (empty) |
| `logicAppsMigrationAssistant.azure.resourceGroup` | Target resource group | `integration-migration-tool-test-rg` |
| `logicAppsMigrationAssistant.azure.location` | Azure region | `eastus` |
| `logicAppsMigrationAssistant.deploymentModel` | Deployment model | `workflow-service-plan` |

## Deploy to Azure

To deploy the migrated workflows:

1. Make sure you've completed Stages 1 through 4 and validated the generated workflows locally.

1. Verify that the deployment settings are configured correctly.

1. Sign in to Azure CLI if you haven't already:

   ```bash
   az login
   ```

1. Proceed with the deployment from the Logic Apps Migration Agent panel.

1. The extension provisions the required resources and deploys your workflows.

## Post-deployment verification

After deployment completes, verify your workflows in Azure:

1. Open the [Azure portal](https://portal.azure.com).

1. Navigate to your Logic Apps Standard resource in the configured resource group.

1. Verify that all workflows appear and are in the **Enabled** state.

1. Test each workflow with sample inputs to confirm they behave as expected in the cloud environment.

1. Check **Application Insights** for any runtime errors or performance issues.

## Deployed resource architecture

The deployment creates the following Azure resources:

| Resource | Description |
|---|---|
| **Logic Apps Standard** (Workflow Service Plan) | Hosts your migrated workflows |
| **Storage Account** | Stores workflow state and run history |
| **Application Insights** | Provides monitoring, logging, and diagnostics |

## Next steps

- [Logic Apps Migration Agent overview](migrate-logic-apps-migration-agent-overview.md)
- [Stage 1 - Discovery: Catalog integration artifacts](migrate-logic-apps-migration-agent-stage-discovery.md)
