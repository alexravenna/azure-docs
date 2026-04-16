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
# Customer intent: As a developer who works with enterprise integration platforms, such as BizTalk Server, Mulesoft, or others, I want to learn how to automate solution migration to Standard workflows in Azure Logic Apps by using the Migration Agent extension in Visual Studio Code.
---

# Migration from enterprise integration platforms to Azure Logic Apps

[!INCLUDE [logic-apps-sku-standard](../includes/logic-apps-sku-standard.md)]

If your organization uses integration platforms like BizTalk Server, MuleSoft Anypoint, or other middleware, modernizing or migrating these workloads by moving to the cloud can prove complex and challenging. A typical migration involves the following tasks:

- Discover and catalog integration artifacts in the source platform.
- Analyze complexity and plan a migration roadmap.
- Convert source artifacts into Standard workflows for Azure Logic Apps.
- Validate generated workflows against source specifications.
- Deploy the migrated solution to Azure.

To guide you through this process, use the Azure Logic Apps Migration Agent in Visual Studio Code. This AI-powered extension automates migration for your enterprise integration solutions to Standard workflows in Azure Logic Apps. The migration agent walks you through a structured 5-stage migration workflow. Built on GitHub Copilot and the Visual Studio Code Language Model API, the extension works with specialized Copilot agents and built-in parsers, while you stay in control at every step.

This article provides an overview about the migration agent, the extension's key capabilities, supported source platforms, and the guided 5-stage migration workflow.

## Supported source and target platforms

The following table lists the source integration platforms that the migration agent currently supports:

| Source | Status | Parser |
|--------|--------|--------|
| BizTalk Server (2016, 2020) | Fully implemented | Built-in |
| MuleSoft Anypoint (Mule 3, Mule 4) | In progress, not yet available | Built-in (stub) |

The Migration Agent is an open-source, extensible project. To add support for a new platform, contribute a built-in parser or create an external parser extension. For more information, see [Add a custom parser for a new platform](migrate-logic-apps-migration-agent-custom-parsers.md).

The following table lists the target platforms and hosting options that the migration agent current supports for generating Standard workflows:

| Target | Hosting option |
|--------|----------------|
| Single-tenant Azure Logic Apps | Workflow Service Plan |
| Your own partially-connected, on-premises infrastructure | Hybrid |

For more information, see [Differences between Standard and Consumption logic apps](../single-tenant-overview-compare.md).

## Key capabilities at a glance

- **Multi-platform support**: Built-in parsers for BizTalk and MuleSoft, plus an extensible parser plugin system for partner platforms.
- **5-stage guided workflow**: A structured migration process from Discovery through Deployment, with progress tracking and visualization at each stage.
- **AI-powered analysis and conversion**: Three specialized Copilot agents (`@migration-analyser`, `@migration-planner`, `@migration-converter`) that analyze, plan, and convert your integration artifacts.
- **Built-in parsers**: TypeScript-based parsers for BizTalk orchestrations, maps, schemas, pipelines, and bindings.
- **Flow visualization**: Interactive architecture diagrams, message flows, gap analysis, and dependency tracking.
- **Azure deployment**: Direct deployment configuration from within VS Code.

## Migration stages

The extension guides you through a 5-stage migration workflow:

:::image type="content" source="./media/migrate-logic-apps-migration-agent/migration-stages.png" alt-text="Diagram that shows the five migration stages: Discovery, Planning, Conversion, Validation, and Deployment.":::

| Stage | Description |
|---|---|
| **Discovery** | Find and catalog all integration artifacts from the source platform. The extension auto-detects the platform, scans files, and builds an artifact inventory and dependency graph. |
| **Planning** | Analyze complexity, plan the migration roadmap, and map source patterns to Logic Apps patterns. The extension generates per-flow migration plans with action mappings, gap analysis, and effort estimates. |
| **Conversion** | Transform source artifacts into Logic Apps Standard workflows, connections, and supporting files. The extension executes task plans generated during planning. |
| **Validation** | Test generated workflows and validate behavior against source specifications. |
| **Deployment** | Deploy generated Logic Apps artifacts to Azure. |

## AI agents

The Logic Apps Migration Agent provisions three GitHub Copilot agents into your workspace:

| Agent | Purpose |
|---|---|
| `@migration-analyser` | Analyzes discovered artifacts, detects flow groups, and generates architecture visualizations. |
| `@migration-planner` | Creates per-flow migration plans with action mappings and gap analysis. |
| `@migration-converter` | Executes conversion tasks to generate Logic Apps workflows and connections. |

These agents use **25 Language Model tools** registered with VS Code to read artifacts, store results, and manage the migration workflow.

## Flow visualization

The Flow Visualizer provides an interactive webview for exploring your integration architecture, including the following views:

| View | Description |
|---|---|
| **Architecture diagram** | System architecture diagram with all artifacts and connections, rendered as Mermaid diagrams. |
| **Message flow** | Per-artifact message flow from trigger to completion. |
| **Components** | Component inventory with details such as adapters, endpoints, and pipelines. |
| **Gap analysis** | Features that have no direct Logic Apps equivalent, with suggested resolutions. |
| **Patterns** | Detected integration patterns such as publish/subscribe, request-reply, and batch processing. |

## Prerequisites

| Requirement | Purpose |
|---|---|
| VS Code 1.85.0 or later | Runtime host |
| [Azure Logic Apps (Standard)](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurelogicapps) extension | Required extension dependency |
| [Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) extension | Local functions runtime and development tasks |
| GitHub Copilot subscription | AI-powered analysis, planning, and conversion |
| [Docker Desktop](https://docs.docker.com/desktop/setup/install/windows-install/) | Local connector resource deployment for testing |
| Azure CLI | Azure resource provisioning and deployment |
| Azure subscription | Deployment to Azure (Stage 5) |

## Extension settings

Configure the extension in VS Code at **Settings** > **Extensions** > **Logic Apps Migration Agent**:

| Setting | Description | Default |
|---|---|---|
| `logicAppsMigrationAgent.deploymentModel` | Target deployment model for Logic Apps Standard | `workflow-service-plan` |
| `logicAppsMigrationAgent.azure.subscriptionId` | Azure subscription ID for deployment | (empty) |
| `logicAppsMigrationAgent.azure.resourceGroup` | Azure resource group for provisioning and testing | `integration-migration-tool-test-rg` |
| `logicAppsMigrationAgent.azure.location` | Azure region for provisioning resources | `i.e. eastus` |

## Next steps

- [Quickstart: Migrate an integration project using the Logic Apps Migration Agent](migrate-logic-apps-migration-agent-quickstart.md)
- [Add a custom parser for a new platform](migrate-logic-apps-migration-agent-custom-parsers.md)
