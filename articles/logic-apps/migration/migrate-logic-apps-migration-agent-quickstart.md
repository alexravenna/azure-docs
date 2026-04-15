---
title: "Quickstart: Migrate Integration workloads using the Logic Apps Migration Agent"
titleSuffix: Azure
description: Learn how to migrate a BizTalk or MuleSoft integration project to Azure Logic Apps Standard using the Logic Apps Migration Agent in VS Code.
author: haroldcampos
author: haroldcampos
ms.author: hcampos
ms.reviewer: estfan
ms.topic: overview
ms.date: 04/06/2026
---

# Quickstart: Migrate integration workloads using the Logic Apps Migration Agent

This quickstart shows you how to use the Logic Apps Migration Agent to migrate an integration workload from BizTalk Server to Azure Logic Apps Standard. You install the extension, open your source project, and walk through all five migration stages: Discovery, Planning, Conversion, Validation, and Deployment.

## Prerequisites

Before you begin, make sure you have the following resources:

- VS Code 1.85.0 or later
- [Azure Logic Apps (Standard)](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurelogicapps) extension installed in VS Code
- [Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) extension installed in VS Code
- An active GitHub Copilot subscription
- [Docker Desktop](https://docs.docker.com/desktop/setup/install/windows-install/) installed and running
- Azure CLI installed
- An Azure subscription
- A BizTalk Server project folder (`.btproj`, `.odx`, `.btm`, `.xsd`, `.btp` files) or a MuleSoft Anypoint project folder

## Install the extension

1. Open VS Code. While not required, we recommend you to open VSCode from where all your integration project directories exist. For instance **C:\Migration>code .** 

:::image type="content" source="./media/migrate-logic-apps-migration-agent/logic-apps-migration-path.png" alt-text="Screenshot that shows the integration directories.":::

1. Go to the **Extensions** view by selecting the Extensions icon in the Activity Bar, or press **Ctrl+Shift+X**.

1. Search for **Logic Apps Migration Agent**.

   :::image type="content" source="./media/migrate-logic-apps-migration-agent/logic-apps-migration-search.png" alt-text="Screenshot that shows the Logic Apps Migration Agent icon in the VS Code Activity Bar.":::
 

1. Select **Install**.

After installation, a new **Logic Apps Migration Agent** icon appears in the Activity Bar on the left side of VS Code.

## Select a source folder

1. Select the **Logic Apps Migration Agent** icon in the Activity Bar.

   :::image type="content" source="./media/migrate-logic-apps-migration-agent/activity-bar-icon.png" alt-text="Screenshot that shows the Logic Apps Migration Agent icon in the VS Code Activity Bar.":::

1. Open a folder that contains your BizTalk, MuleSoft or other project in VS Code, by selecting **File** > **Open Folder**.

1. When prompted, select the source folder that contains your integration artifacts. Alternatively, you can open the Command Palette (**Ctrl+Shift+P**) and run **Logic Apps Migration Agent: Select Source Folder**.

   :::image type="content" source="./media/migrate-logic-apps-migration-agent/logic-apps-migration-dialog.png" alt-text="Screenshot that shows the Logic Apps Migration Agent folder selection.":::
 

The extension auto-detects the source platform and begins the migration workflow.

## Stage 1: Discovery

The first stage discovers and catalogs all integration artifacts in your source project.

:::image type="content" source="./media/migrate-logic-apps-migration-agent/discovery-stage.png" alt-text="Screenshot that shows the Discovery stage in the Logic Apps Migration Agent, displaying discovered artifacts and dependencies.":::

During discovery, the extension performs the following actions:

1. **Detects the source platform** from file patterns (for example, `.btproj` files for BizTalk).

1. **Scans all source files** using the built-in parser for your platform.

1. **Builds an artifact inventory** that includes orchestrations, schemas, maps, pipelines, and bindings.

1. **Generates a dependency graph** that shows how artifacts relate to each other.

After the scan completes, the `@migration-analyser` Copilot agent analyzes the discovered artifacts and detects logical flow groups (sets of artifacts that work together). Something to consider is that the generated flows sometimes will not necessarily reflect a 1:1 relationship with the legacy integration applications. The agent will infer the flows that best reflect the legacy system intgration artifacts (i.e BizTalk workloads) in Logic Apps Standard. If you want to change the flows to reflect a 1:1 mapping with your integration workloads, you can use GHCP and indicate flows need to map your BizTalk applications. This is one of the first paradigm changes in the modernization. What was optimum for BizTalk will not necessarily be for Logic Apps Standard.

:::image type="content" source="./media/migrate-logic-apps-migration-agent/discovery-stage-detail.png" alt-text="Screenshot that shows the Discovery stage details in the Logic Apps Migration Agent, displaying discovered artifacts and dependencies.":::

To continue with the next stage, you need to select the **Analyze Source Design** button in any of the flows. This will:

- Generate architecture diagrams and message flows and components.
- Identify missing dependencies.
- Perform a Gap analysis for features and propose Logic Apps or other services alternatives.
- Detect integration patterns such as publish/subscribe, request-reply, and batch.
- Generate a discovery report based on the findings.


Review the discovery results in the **Flow Visualizer** panel, which provides interactive tabs for architecture, message flows, components, gaps, and patterns. There are three important selections in this panel:

1. **Suggest a change** to ask direct changes to the analysis.

1. **Regenerate analysis** to regenerate analysis after making a change. For instance adding a missing dependency, artifact, or specifications.

1. **Export Report** to generate a report with the outcome of the discovery.

:::image type="content" source="./media/migrate-logic-apps-migration-agent/discovery-stage-analysis.png" alt-text="Screenshot that shows the Discovery stage details in the Logic Apps Migration Agent, displaying discovered artifacts and dependencies.":::

> [!TIP]
> Use the Copilot chat integration in the Flow Visualizer to discuss or correct any flow group. Select a flow group and ask the `@migration-analyser` agent questions about the detected architecture. You can also provide information about any missing gaps and Regenerate the analysis.

> [!TIP]
> To continue analysing flows, select the Home tab, or select **Home Page**.

## Stage 2: Planning

To continue with the Planning stage, you need to select the **Plan LogicApp Design** button in any of the flows. The planning stage creates a migration roadmap.

:::image type="content" source="./media/migrate-logic-apps-migration-agent/planning-stage-main.png" alt-text="Screenshot that shows the Planning stage with per-flow migration plans and action mappings.":::

The `@migration-planner` agent generates per-flow migration plans that include:

- **Architecture**: Designer view (including Code view{}) and Architecture diagram for the proposed solution.
- **Aditional Azure components**: This includes explicit and non-explicit conversions for the proposed design.
- **Operation mappings**: One-to-one mappings from source platform components to their Logic Apps Standard equivalents.
- **Artifacts disposition**: Artifacts requiring conversion along with their Upload destinations.
- **Migration GAPS**: Components that don't have direct equivalents, along with recommended workarounds.
- **Integration patterns**: Detected patterns for the integration flow.
- **Summary**: High level overview of the proposed workflow.

- **Effort estimates**: Estimated complexity and effort for each flow.
- **Task plans**: Step-by-step conversion tasks for the next stage.

Review each plan carefully. You can interact with the `@migration-planner` agent via Copilot chat to adjust plans, ask questions about specific mappings, or request alternative approaches. To continue with the conversion, select **Home Page** or return to the **Home** tab

:::image type="content" source="./media/migrate-logic-apps-migration-agent/planning-stage-create.png" alt-text="Screenshot that shows the Planning stage with per-flow migration plans and action mappings.":::

## Stage 3: Create (Conversion Tasks)

The conversion stage transforms your source artifacts into Logic Apps Standard workflows.

:::image type="content" source="./media/migrate-logic-apps-migration-agent/conversion-stage-main.png" alt-text="Screenshot that shows the Conversion stage generating Logic Apps Standard workflow files.":::

The `@migration-converter` agent executes the task plans from the planning stage:

1. **Scaffolds the Logic Apps project** structure, including `host.json`, `local.settings.json`, and folder layout.

1. **Generates `workflow.json` files** for each flow, using Logic Apps Standard workflow definitions.

1. **Creates `connections.json`** with the required connector configurations.

1. **Generates .NET local functions** when custom logic is needed that isn't available through built-in connectors.

1. **Validates completeness** to ensure no stubs or placeholder code remain in the generated output.

> [!NOTE]
> The conversion process produces complete, ready-to-run workflow definitions. The `@migration-converter` agent uses the `no-stubs-code-generation` skill to ensure that all generated code is fully functional.

## Stage 4: Validation

The validation stage tests the generated workflows against the source specifications.

Use this stage to:

- Run generated workflows locally using the Azure Functions runtime and Docker Desktop.
- Identify and fix any discrepancies.
- Compare the behavior of the generated workflows with the original integration flows. The Validation stage provides the ability to **bring your own test cases and specifications**. This steps is key to validate your integration workload performs as expected.

:::image type="content" source="./media/migrate-logic-apps-migration-agent/validation-stage-blackbox.png" alt-text="Screenshot that shows the Validation stage with Balck box testing.":::

The extension provides runtime validation and testing guidance through the `@migration-converter` agent. You can verify the generated solution in the out directory.

:::image type="content" source="./media/migrate-logic-apps-migration-agent/deployment-stage-outcome.png" alt-text="Screenshot that shows the path of the generated code.":::

## Stage 5: Deployment

The final stage deploys your migrated Logic Apps artifacts to Azure. Before deployment, configure the following extension settings at **Settings** > **Extensions** > **Logic Apps Migration Agent**:

- **Azure Subscription ID**: Your Azure subscription for deployment.
- **Resource Group**: The target resource group for provisioning.
- **Location**: The Azure region for your resources.

:::image type="content" source="./media/migrate-logic-apps-migration-agent/deployment-stage-main.png" alt-text="Screenshot that shows the end to end testing with deployment in target environment.":::

The extension uses the Azure CLI to provision the required infrastructure and deploy your Logic Apps Standard workflows. The following is a complete solution.

:::image type="content" source="./media/migrate-logic-apps-migration-agent/deployment-stage-final.png" alt-text="Screenshot that shows the end to end testing with deployment in target environment.":::


## Reset the migration

If you need to start over, open the Command Palette (**Ctrl+Shift+P**) and run **Logic Apps Migration Agent: Reset Migration**. This command clears all migration state and lets you begin again from the Discovery stage.

## Next steps

- [Logic Apps Migration Agent overview](migrate-logic-apps-migration-agent-overview.md)
- [Add a custom parser for a new platform](migrate-logic-apps-migration-agent-custom-parsers.md)
