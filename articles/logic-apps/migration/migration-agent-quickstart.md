---
title: Quickstart - Migrate Workloads from Integration Platforms
titleSuffix: Azure Logic Apps
description: Migrate BizTalk Server or MuleSoft integration projects to Standard workflows in Azure Logic Apps by using the migration agent in Visual Studio Code.
services: azure-logic-apps
ms.suite: integration
author: haroldcampos
ms.author: hcampos
ms.reviewers: estfan, azla
ms.topic: how-to
ai-usage: ai-assisted
ms.update-cycle: 180-days
ms.date: 04/21/2026
# Customer intent: As a developer who works with enterprise integration platforms, such as BizTalk Server, Mulesoft, or others, I want to learn how to quickly automate the migration process for my integration project to Standard workflows in Azure Logic Apps by using the Migration Agent extension in Visual Studio Code.
---

# Quickstart: Migrate integration projects to Standard workflows in Azure Logic Apps (preview)

[!INCLUDE [logic-apps-sku-standard](../includes/logic-apps-sku-standard.md)]

> [!NOTE]
>
> This preview feature is subject to the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

When your team needs to migrate workloads from legacy platforms like BizTalk Server to the cloud, you'll likely find the process complex, time-consuming, and challenging. To help simplify and ease this task, the Azure Logic Apps Migration Agent in Visual Studio Code automates this process through five guided stages.

This quickstart shows how to migrate an example integration workload from BizTalk Server to Standard workflows in Azure Logic Apps by using the Azure Logic Apps Migration Agent in Visual Studio Code. You learn how to install the extension, open your source project, and follow the agent as it walks you through the migration stages: Discovery, Planning, Conversion, Validation, and Deployment.

> [!NOTE]
>
> Although the migration agent almost runs autonomously, it might prompt you to allow running specific commands for required tasks. To let the agent continue, select **Allow**.

For more information, see [About Azure Logic Apps Migration Agent](migration-agent-overview.md).

## Prerequisites

Before you start, meet the following requirements:

| Requirement | Purpose |
|-------------|---------|
| [Azure subscription - Get a free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) | Deployment to Azure (Stage 5) |
| [Azure CLI](/cli/azure/install-azure-cli) | Azure resource provisioning and deployment |
| [Visual Studio Code 1.85.0 or later](https://code.visualstudio.com/download) | Runtime host |
| [Azure Logic Apps Migration Agent extension]() | Required extension with migration agent |
| [Azure Logic Apps (Standard) extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurelogicapps) | Required extension dependency |
| [Azure Functions extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) | Local functions runtime and development tasks |
| [GitHub Copilot subscription](https://github.com/features/copilot/plans) | AI-powered analysis, planning, and conversion |
| [Docker Desktop](https://docs.docker.com/desktop/setup/install/windows-install/) | Local connector resource deployment for testing |
| Folder with BizTalk Server or MuleSoft Anypoint projects | Folder that contains integration project folders with source artifacts and files. For example, a BizTalk project folder includes files with the *.btproj*, *.odx*, *.btm*, *.xsd*, and *.btp* file name extensions. |

### 1: Install the Migration Agent extension

1. Open Visual Studio Code.

   Optionally, but recommended, open Visual Studio Code from the folder or directory where your integration projects exist, for example, **C:\Migration\<*project-folders*>**.

   :::image type="content" source="./media/migration-agent-quickstart/migration-path.png" alt-text="Screenshot that shows the folder or directory with all integration project folders.":::

1. On the Activity Bar, select **Extensions**. (Keyboard: Ctrl + Shift + X)

1. In the **Extensions: Marketplace** search box, find the **Azure Logic Apps Migration Agent** extension, and select **Install**.

   :::image type="content" source="./media/migration-agent-quickstart/migration-search.png" alt-text="Screenshot that shows Visual Studio Code, Extensions Marketplace, and the Azure Logic Apps Migration Agent extension.":::

   After installation completes, the Activity Bar shows the icon for the **Azure Logic Apps Migration Agent** (![Icon for Azure Logic Apps Migration Agent.](media/migration-agent-quickstart/migration-agent-icon.png)).

 ### 2: Select your source folder

1. In Visual Studio Code, on the Activity Bar, select the **Azure Logic Apps Migration Agent** icon (![Icon for Azure Logic Apps Migration Agent.](media/migration-agent-quickstart/migration-agent-icon.png)).

1. In the **Azure Logic Apps Migration Agent** window, in the **Discovery Results** section, choose **Select Source Folder**.

   > [!TIP]
   >
   > From the Command Palette (Keyboard Ctrl + Shift + P), find and run **Azure Logic Apps Migration Agent: Select Source Folder**.

1. Find and select the source folder that contains your BizTalk, MuleSoft, or other integration projects, then select **Select Source Project Folder or MSI**.

   :::image type="content" source="./media/migration-agent-quickstart/migration-dialog.png" alt-text="Screenshot that shows Visual Studio Code with the Azure Logic Apps Migration Agent and the source folder with projects.":::

   The extension automatically detects the source platform and begins the migration workflow, starting with the discovery stage.

1. Follow the agent as it walks you through each migration stage, starting with the Discovery stage.

## Stage 1: Discovery

In this stage, the migration agent finds and catalogs the integration artifacts in your source project. During the discovery stage, the migration agent performs the following actions in the described order with occasional input from you.

### Step 1: Detect the source platform

The migration agent determines your source platform, based on file patterns, such as BizTalk Server *.btproj* files.

The following screenshot shows the identified platform and example found artifacts and dependencies:

:::image type="content" source="./media/migration-agent-quickstart/discovery-stage.png" alt-text="Screenshot that shows the Azure Logic Apps Migration Agent extension and the Discovery stage with the found artifacts and dependencies.":::

### Step 2: Scan source files

The migration agent scans the found source files by using the built-in parser for your platform. After the scan completes, the `@migration-analyser` Copilot agent analyzes the discovered artifacts and detects logical flow groups, which are sets of artifacts that work together.

The following screenshot shows how each example integration project maps to a logical flow group:

:::image type="content" source="./media/migration-agent-quickstart/discovery-stage-detail.png" alt-text="Screenshot that shows the Discovery stage details with the found artifacts and dependencies.":::
   
Generated logical flows don't always reflect a 1:1 relationship with legacy integration applications. The migration agent infers the flows that best reflect the legacy system's intgration artifacts, such as BizTalk workloads, as Standard workflows in Azure Logic Apps.

> [!TIP]
>
>To edit these logical flows so they map 1:1 to your integration workloads, use GitHub Copilot and specify that flows must map to your BizTalk applications. However, consider that optimal for BizTalk isn't the same as optimal for Standard workflows in Azure Logic Apps. This concept is one of the first paradigm changes in modernization.

### Step 3: Analyze source design

1. After the migration agent completes scanning and shows the resulting logical flow groups, on the **Home** tab, choose a logical flow group, and select **Analyze Source Design**.

   :::image type="content" source="./media/migration-agent-quickstart/analyze-source-design.png" alt-text="Screenshot that shows migration agent home page with Analyze Source Design selected.":::

   The agent performs the following tasks:

   1. Builds an artifact inventory that includes orchestrations, schemas, maps, pipelines, and bindings.

   1. Generates a dependency graph that shows the relationships between artifacts.

      The migration agent runs the following tasks to generate the dependency graph:

      - Generate architecture (Mermaid) diagrams that show message flows and components.
      - Identify missing dependencies.
      - Perform a gap analysis for features.
      - Detect integration patterns such as publish-subscribe, request-reply, and batch.
      - Propose mappings for Azure Logic Apps or other services alternatives.
      - Generate a discovery report based on the findings.

      After the migration agent successfully generates the dependency graph, the flow visualizer opens and shows the following interactive tabs:
   
      - **Architecture Diagram**
      - **Mesage Flow**
      - **Components**
      - **Missing Dependencies**
      - **Gap Analysis**
      - **Patterns**
      - **Learn BizTalk**

      :::image type="content" source="./media/migration-agent-quickstart/discovery-stage-analysis.png" alt-text="Screenshot that shows theflow visualizer with the results from the Discovery stage.":::

1. To review the analysis results, select a tab to review the related information.

### Step 4: Update or export the analysis

1. After you review the analysis results, on the flow visualizer title bar, select one of the following actions:

   | Action | Description |
   |--------|-------------|
   | **Suggest a Change** | Request direct changes to the analysis. <br><br>**Tip**: To discuss potential updates or corrections to any flow group, in the flow visualizer, use the Copilot chat integration. Select a flow group and ask the `@migration-analyser` agent questions about the detected architecture. Provide information about any missing gaps, and then regenerate the analysis. |
   | **Regenerate Analysis** | After you update the analysis, such as add a missing dependency, artifact, or specification, rerun the analysis. |
   | **Export Report** | Generate a report with the discovery results in a shareable format. |

   Or, to analyze more flows, select the **Home** tab or the home page icon.

1. When you finish, continue to the next section for the Planning stage.

## Stage 2: Planning

After you finish your analysis, start the Planning stage by creating a migration roadmap to follow. 

1. On the **Home** tab, choose the logical flow group you want, and select **Plan Logic App Design**.

   :::image type="content" source="./media/migration-agent-quickstart/plan-logic-app-design.png" alt-text="Screenshot that shows migration agent home page with Plan Logic App Design selected.":::

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

   :::image type="content" source="./media/migration-agent-quickstart/planning-stage-main.png" alt-text="Screenshot that shows the Planning stage with per-flow migration plans and action mappings.":::


Review each plan carefully. You can interact with the `@migration-planner` agent via Copilot chat to adjust plans, ask questions about specific mappings, or request alternative approaches. To continue with the conversion, select **Home Page** or return to the **Home** tab

:::image type="content" source="./media/migration-agent-quickstart/planning-stage-create.png" alt-text="Screenshot that shows the Planning stage with per-flow migration plans and action mappings.":::

## Stage 3: Create (Conversion Tasks)

The conversion stage transforms your source artifacts into Logic Apps Standard workflows.

:::image type="content" source="./media/migration-agent-quickstart/conversion-stage-main.png" alt-text="Screenshot that shows the Conversion stage generating Logic Apps Standard workflow files.":::

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

:::image type="content" source="./media/migration-agent-quickstart/validation-stage-blackbox.png" alt-text="Screenshot that shows the Validation stage with Balck box testing.":::

The extension provides runtime validation and testing guidance through the `@migration-converter` agent. You can verify the generated solution in the out directory.

:::image type="content" source="./media/migration-agent-quickstart/deployment-stage-outcome.png" alt-text="Screenshot that shows the path of the generated code.":::

## Stage 5: Deployment

The final stage deploys your migrated Logic Apps artifacts to Azure. Before deployment, configure the following extension settings at **Settings** > **Extensions** > **Logic Apps Migration Agent**:

- **Azure Subscription ID**: Your Azure subscription for deployment.
- **Resource Group**: The target resource group for provisioning.
- **Location**: The Azure region for your resources.

:::image type="content" source="./media/migration-agent-quickstart/deployment-stage-main.png" alt-text="Screenshot that shows the end to end testing with deployment in target environment.":::

The extension uses the Azure CLI to provision the required infrastructure and deploy your Logic Apps Standard workflows. The following is a complete solution.

:::image type="content" source="./media/migration-agent-quickstart/deployment-stage-final.png" alt-text="Screenshot that shows the end to end testing with deployment in target environment.":::

## Reset the migration

If you need to start over, open the Command Palette (**Ctrl+Shift+P**) and run **Logic Apps Migration Agent: Reset Migration**. This command clears all migration state and lets you begin again from the Discovery stage.

## Next steps

- [Add a custom parser for a new platform](migration-agent-custom-parsers.md)
