---
title: Logic Apps Migration Agent for Enterprise Integration
titleSuffix: Azure
description: Provides an overview of the Logic Apps Migration Agent, a VS Code extension that automates migration from BizTalk, MuleSoft, and other integration platforms to Azure Logic Apps Standard.
author: haroldcampos
ms.author: hcampos
ms.reviewer: estfan
ms.topic: overview
ms.date: 04/06/2026
---

# Logic Apps Migration Agent for enterprise integration

This article describes the Logic Apps Migration Agent, which is an AI-powered VS Code extension that delivers end-to-end support for migrating enterprise integration solutions to Azure Logic Apps Standard.

Enterprises often rely on legacy integration platforms such as BizTalk Server, MuleSoft Anypoint, and other middleware to connect their systems, applications, and data. Modernizing these integration workloads to the cloud involves:

- Discovering and cataloging all integration artifacts from the source platform
- Analyzing complexity and planning the migration roadmap
- Converting source artifacts into Azure Logic Apps Standard workflows
- Validating generated workflows against source specifications
- Deploying the migrated solution to Azure

Built on **GitHub Copilot** and the **VS Code Language Model API**, the Logic Apps Migration Agent provides a structured 5-stage migration workflow with AI-powered analysis and conversion. The extension uses specialized Copilot agents and built-in parsers to automate the migration process while keeping you in control at every step.

## Supported source platforms

| Platform | Status | Parser |
|---|---|---|
| **BizTalk Server** (2016, 2020) | Fully implemented | Built-in |
| **MuleSoft Anypoint** (Mule 3, Mule 4) | In progress | Built-in (stub) |

The Logic Apps Migration Agent is an open-source, extensible project. To add support for a new platform, you can contribute a built-in parser or create an external parser extension. For more information, see [Add a custom parser for a new platform](migrate-logic-apps-migration-agent-custom-parsers.md).

## Target platform

- **Azure Logic Apps Standard** (Azure)
- **Azure Logic Apps Standard** (On-premises - Hybrid deployment model)

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
