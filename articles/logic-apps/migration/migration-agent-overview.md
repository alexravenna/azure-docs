---
title: Migration from Other Integration Platforms
titleSuffix: Azure Logic Apps
description: Learn about automating migration to Standard workflows in Azure Logic Apps from BizTalk Server, MuleSoft, and other integration platforms by using the Migration Agent extension in Visual Studio Code.
services: azure-logic-apps
ms.suite: integration
author: haroldcampos
ms.author: hcampos
ms.reviewers: estfan, azla
ms.topic: overview
ai-usage: ai-assisted
ms.update-cycle: 180-days
ms.date: 04/21/2026
# Customer intent: As a developer who works with enterprise integration platforms, such as BizTalk Server, MuleSoft, or others, I want to learn about automating the migration process for my integration project to Standard workflows in Azure Logic Apps by using the Migration Agent extension in Visual Studio Code.
---

# Migration from enterprise integration platforms to Azure Logic Apps (preview)

[!INCLUDE [logic-apps-sku-standard](../includes/logic-apps-sku-standard.md)]

> [!NOTE]
>
> This preview feature is subject to the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

If your organization uses integration platforms like BizTalk Server, MuleSoft Anypoint, or other middleware, modernizing or migrating these workloads by moving to the cloud can prove complex and challenging. A typical migration involves the following tasks:

- Discover and catalog integration artifacts in the source platform.
- Analyze complexity and plan a migration roadmap.
- Convert source artifacts into Standard workflows for Azure Logic Apps.
- Validate generated workflows against source specifications.
- Deploy the migrated solution to Azure.

To guide you through this process, use the Azure Logic Apps Migration Agent in Visual Studio Code. This AI-powered extension automates migration for your enterprise integration solutions to Standard workflows in Azure Logic Apps. The migration agent walks you through a structured 5-stage migration workflow. Built on GitHub Copilot and the Visual Studio Code Language Model API, the extension works with specialized Copilot agents and built-in parsers, while you stay in control at every step.

This article provides an overview about the migration agent, the extension's key capabilities, supported source platforms, and the guided 5-stage migration workflow.

## Supported source platforms and target environments

The migration agent currently supports the following source integration platforms:

| Source platform | Status | Parser |
|-----------------|--------|--------|
| BizTalk Server (2016, 2020) | Fully completed | Built-in |
| MuleSoft Anypoint (Mule 3, Mule 4) | In progress, not yet available | Built-in (stub) |

Migration Agent is an open-source, extensible project. To add support for a new platform, contribute a built-in parser or create an external parser extension. For more information, see [Add a custom parser for a new platform](migrate-logic-apps-migration-agent-custom-parsers.md).

The migration agent currently generates Standard workflows for the following target deployment environments and hosting options:

| Target environment | Hosting option |
|--------------------|----------------|
| Single-tenant Azure Logic Apps | Workflow Service Plan |
| Your own partially connected, on-premises infrastructure | Hybrid |

For more information, see [Differences between Standard and Consumption logic apps](../single-tenant-overview-compare.md).

## Key capabilities

The migration agent includes the following core capabilities:

| Capability | Description |
|------------|-------------|
| Multi-platform support | Built-in parsers for BizTalk and MuleSoft, plus an extensible parser plug-in system for partner platforms. |
| 5-stage guided workflow | A structured migration process from discovery to deployment, including progress tracking and visualization at each stage. |
| AI-powered analysis and conversion | Specialized Copilot agents that analyze, plan, and convert your integration artifacts: <br><br>- `@migration-analyser` <br>- `@migration-planner` <br>- `@migration-converter` |
| Built-in parsers | TypeScript-based parsers for BizTalk orchestrations, maps, schemas, pipelines, and bindings. |
| Flow visualization | Interactive architecture diagrams, message flows, gap analysis, and dependency tracking. |
| Azure deployment | Direct deployment configuration from inside Visual Studio Code. |

## Migration stages

The migration agent guides you through the following 5-stage migration workflow:

:::image type="content" source="./media/migrate-logic-apps-migration-agent/migration-stages.png" alt-text="Diagram that shows the five migration stages: Discovery, Planning, Conversion, Validation, and Deployment.":::

| Stage | Purpose |
|-------|---------|
| **Discovery** | Find and catalog integration artifacts on the source platform. <br><br>The agent automatically detects the platform, scans files, and builds an artifact inventory and dependency graph. |
| **Planning** | Analyze complexity, plan the migration roadmap, and map source patterns to Logic Apps patterns. <br><br>The agent generates migration plans for each flow with action mappings, gap analysis, and effort estimates. |
| **Conversion** | Transform source artifacts into Standard workflows, connections, and supporting files for Azure Logic Apps. <br><br>The agent executes the task plans generated during the planning stage. |
| **Validation** | Test generated workflows and validate behavior against source specifications. |
| **Deployment** | Deploy generated artifacts for Azure Logic Apps to Azure. |

## AI agents

In your Visual Studio Code project workspace, the migration agent sets up and works with the following GitHub Copilot agents:

| Agent | Task |
|-------|------|
| `@migration-analyser` | Analyze discovered artifacts, detects flow groups, and generates architecture visualizations. |
| `@migration-planner` | Create migration plans for each flow with action mappings and gap analysis. |
| `@migration-converter` | Run conversion tasks that generate Standard workflows and connections for Azure Logic Apps. |

These agents work with 25 language model tools registered in Visual Studio Code to read artifacts, store results, and manage the migration workflow.

## Flow visualization

Explore your integration architecture through an interactive browser by using the flow visualizer, which includes the following views:

| View | Shows |
|------|-------|
| Architecture diagram | System architecture diagram with all artifacts and connections, rendered as a [Mermaid diagram](https://mermaid.ai/open-source). |
| Message flow | Per-artifact message flows from trigger to completion. |
| Components | Components inventory with details such as adapters, endpoints, and pipelines. |
| Gap analysis | Summary about features without a direct equivalent in Azure Logic Apps, including suggested resolutions. |
| Patterns | Detected integration patterns such as publish-subscribe, request-reply, and batch processing. |

## Prerequisites

To use the Azure Logic Apps Migration Agent extension in Visual Studio Code, meet the following requirements:

| Requirement | Purpose |
|-------------|---------|
| [Azure subscription - Get a free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) | Deployment to Azure (Stage 5) |
| [Azure CLI](/cli/azure/install-azure-cli) | Azure resource provisioning and deployment |
| [Visual Studio Code 1.85.0 or later](https://code.visualstudio.com/download) | Runtime host |
| [Azure Logic Apps (Standard) extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurelogicapps) | Required extension dependency |
| [Azure Functions extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) | Local functions runtime and development tasks |
| [GitHub Copilot subscription](https://github.com/features/copilot/plans) | AI-powered analysis, planning, and conversion |
| [Docker Desktop](https://docs.docker.com/desktop/setup/install/windows-install/) | Local connector resource deployment for testing |

## Extension settings

The Azure Logic Apps Migration Agent extension in Visual Studio Code provides the following settings that you can update:

| Setting | Description | Default | Options or examples |
|---------|-------------|---------|---------------------|
| `logicAppsMigrationAgent.deploymentModel` | The target deployment model for Azure Logic Apps (Standard) | `workflow-service-plan` | `hybrid` |
| `logicAppsMigrationAgent.azure.subscriptionId` | The Azure subscription ID for deployment | (empty) | Not applicable |
| `logicAppsMigrationAgent.azure.resourceGroup` | The Azure resource group for provisioning and testing | (empty) | Example: `integration-migration-tool-test-rg` |
| `logicAppsMigrationAgent.azure.location` | The Azure region for provisioning resources | (empty) | Example: `eastus` |

To view these settings, from the **File** menu, go to **Preferences** > **Settings** > **Extensions** > **Logic Apps Migration Agent**.

## Next steps

> [!div class="nextstepaction"]
> [Quickstart: Migrate an integration project using the Logic Apps Migration Agent](migrate-logic-apps-migration-agent-quickstart.md)
