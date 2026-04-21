---
title: Quickstart - Migrate from Integration Platforms
description: Automate migration for BizTalk Server, MuleSoft integration projects to Azure Logic Apps (Standard) by using the Migration Agent in Visual Studio Code.
services: azure-logic-apps
ms.suite: integration
author: haroldcampos
ms.author: hcampos
ms.reviewers: estfan, azla
ms.topic: how-to
ai-usage: ai-assisted
ms.update-cycle: 365-days
ms.date: 04/27/2026
# Customer intent: As a developer who works with enterprise integration platforms, such as BizTalk Server, MuleSoft, and others, I want to learn how to quickly automate the migration process for my integration project to Standard workflows in Azure Logic Apps by using the Migration Agent extension in Visual Studio Code.
---

# Quickstart: Automate migration for integration projects to Azure Logic Apps (Standard) (preview)

[!INCLUDE [logic-apps-sku-standard](../includes/logic-apps-sku-standard.md)]

> [!NOTE]
>
> This preview feature is subject to the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

When your team needs to migrate workloads from legacy platforms like BizTalk Server to the cloud, you might find the process complex, time-consuming, and challenging. To help simplify and ease this task, the Azure Logic Apps Migration Agent in Visual Studio Code automates this process through five guided stages.

This quickstart shows how to migrate an example integration workload from BizTalk Server to Standard workflows in Azure Logic Apps by using the Azure Logic Apps Migration Agent in Visual Studio Code. You learn how to install the extension, open your source project, and follow the agent as it walks you through the migration stages: Discovery, Planning, Conversion, Validation, and Deployment.

> [!NOTE]
>
> Although the migration agent runs almost autonomously, it might prompt you to allow running specific commands for required tasks. To let the agent continue, select **Allow**.

For more information, see [Migration automation from integration platforms to Azure Logic Apps](migration-agent-overview.md).

## Prerequisites

Before you start, make sure you meet the following requirements:

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
| Folder with BizTalk Server projects | Folder that contains integration project folders with source artifacts and files. For example, a BizTalk project folder includes files with the following file name extensions: `.btproj`, `.odx`, `.btm`, `.xsd`, and `.btp`. |

### 1: Install the Migration Agent extension

1. Open Visual Studio Code.

   Optionally, but recommended, open Visual Studio Code from the folder or directory where your integration projects exist, for example, **C:\Migration\\<*project-folders*>**.

   :::image type="content" source="media/migration-agent-quickstart/migration-path.png" alt-text="Screenshot that shows the folder or directory with all integration project folders.":::

1. On the Activity Bar, select **Extensions**. (Keyboard: Ctrl+Shift+X)

1. In the **Extensions: Marketplace** search box, find the **Azure Logic Apps Migration Agent** extension, and select **Install**.

   :::image type="content" source="media/migration-agent-quickstart/migration-search.png" alt-text="Screenshot that shows Visual Studio Code, Extensions Marketplace, and the Azure Logic Apps Migration Agent extension.":::

   After installation completes, the Activity Bar shows the icon for the **Azure Logic Apps Migration Agent** (![Icon for Azure Logic Apps Migration Agent.](media/migration-agent-quickstart/migration-agent-icon.png)).

 ### 2: Select your source folder

1. In Visual Studio Code, on the Activity Bar, select the **Azure Logic Apps Migration Agent** icon (![Icon for Azure Logic Apps Migration Agent.](media/migration-agent-quickstart/migration-agent-icon.png)).

1. In the **Azure Logic Apps Migration Agent** window, in the **Discovery Results** section, choose **Select Source Folder**.

   > [!TIP]
   >
   > To run this action as a command, open the Command Palette (Keyboard: Ctrl+Shift+P). Enter and run **Azure Logic Apps Migration Agent: Select Source Folder**.

1. Find and select the source folder that contains your BizTalk, MuleSoft, or other integration projects, and then select **Select Source Project Folder or MSI**.

   :::image type="content" source="media/migration-agent-quickstart/migration-dialog.png" alt-text="Screenshot that shows Visual Studio Code with the Azure Logic Apps Migration Agent and the source folder with projects.":::

   The extension automatically detects the source platform and begins the migration workflow, starting with the discovery stage.

1. Follow the agent as it walks you through each migration stage, starting with the Discovery stage.

## Migration stage 1: Discovery

In this stage, the migration agent finds and catalogs the integration artifacts in your source project. During the discovery stage, the migration agent performs the following actions in the described order with occasional input from you. For more information, see [Migration agent: Discovery stage](migration-agent-discovery-stage.md).

### Step 1: Detect the source platform

The migration agent determines your source platform, based on file patterns, such as BizTalk Server (`.btproj`) files.

The following screenshot shows the identified platform with example detected artifacts and dependencies:

:::image type="content" source="media/migration-agent-quickstart/discovery-stage.png" alt-text="Screenshot that shows the Azure Logic Apps Migration Agent extension and the Discovery stage with the detected artifacts and dependencies.":::

### Step 2: Scan source files

The migration agent scans the detected source files by using the built-in parser for your platform. After the scan completes, the `@migration-analyser` Copilot agent analyzes the discovered artifacts and detects logical flow groups, which are sets of artifacts that work together.

The following screenshot shows how each example integration project maps to a logical flow group:

:::image type="content" source="media/migration-agent-quickstart/discovery-stage-detail.png" alt-text="Screenshot that shows the Discovery stage details with the detected artifacts and dependencies.":::
   
Generated logical flows don't always reflect a 1:1 relationship with legacy integration applications. The migration agent infers the flows that best reflect the legacy system's integration artifacts, such as BizTalk workloads, as Standard workflows in Azure Logic Apps.

> [!TIP]
>
>To edit these logical flows so they map 1:1 to your integration workloads, use GitHub Copilot and specify that flows must map to your BizTalk applications. However, consider that optimal for BizTalk isn't the same as optimal for Standard workflows in Azure Logic Apps. This concept is one of the first paradigm changes in modernization.

### Step 3: Analyze source design

After the migration agent completes scanning and shows the resulting logical flow groups, follow these steps:

1. On the **Home** tab, for the logical flow group you want, select **Analyze Source Design**, for example:

   :::image type="content" source="media/migration-agent-quickstart/analyze-source-design.png" alt-text="Screenshot that shows migration agent home page with Analyze Source Design selected.":::

   The agent performs the following tasks:

   1. Builds an artifact inventory that includes orchestrations, schemas, maps, pipelines, and bindings.

   1. Generates a dependency graph that shows the relationships between artifacts.

      To generate the dependency graph, the migration agent runs the following tasks:

      - Generates architecture (Mermaid) diagrams that show message flows and components.
      - Identifies missing dependencies.
      - Performs a gap analysis for features.
      - Detects integration patterns such as publish-subscribe, request-reply, and batch.
      - Proposes mappings for Azure Logic Apps or other services alternatives.
      - Generates a discovery report based on the findings.

      After the migration agent successfully generates the dependency graph, the flow visualizer opens and shows the following interactive tabs:
   
      - **Architecture Diagram**
      - **Message Flow**
      - **Components**
      - **Missing Dependencies**
      - **Gap Analysis**
      - **Patterns**
      - **Learn BizTalk**

      :::image type="content" source="media/migration-agent-quickstart/discovery-stage-analysis.png" alt-text="Screenshot that shows the flow visualizer with the results from the Discovery stage.":::

      For more information, see [Flow visualization for integration architecture](migration-agent-overview.md#flow-visualization-for-integration-architecture).

1. To review the analysis results, select a tab to review the related information.

### Step 4: Update or export the analysis

1. After you review the analysis results, on the flow visualizer title bar, select one of the following actions:

   | Action | Description |
   |--------|-------------|
   | **Suggest a Change** | Request direct changes to the analysis. <br><br>**Tip**: To discuss potential updates or corrections to any flow group, in the flow visualizer, use the Copilot chat window. Select a flow group and ask the `@migration-analyser` agent questions about the detected architecture. Provide information about any missing gaps, and then regenerate the analysis. |
   | **Regenerate Analysis** | After you update the analysis, such as add a missing dependency, artifact, or specification, rerun the analysis. |
   | **Export Report** | Generate a report with the discovery results in a shareable format. |

   Or, to analyze more flows, select the **Home** tab or the home page icon.

1. When you finish, go to the next section for the Planning stage.

## Migration stage 2: Planning

After you finish your analysis, start the Planning stage by creating a migration roadmap to follow. 

1. On the **Home** tab, choose the logical flow group you want, and select **Plan Logic App Design**.

   :::image type="content" source="media/migration-agent-quickstart/plan-logic-app-design-stage.png" alt-text="Screenshot that shows migration agent home page with Plan Logic App Design selected.":::

   The `@migration-planner` agent generates a migration plan that usually includes the following sections:

   | Section name | Description |
   |--------------|-------------|
   | **Architecture** | The designer view, code view, and architecture diagram for the proposed solution. |
   | **Additional Azure components** | The explicit and non-explicit Azure component conversions required for the proposed design. |
   | **Operations mapping** | The one-to-one mappings from source platform components to their Standard workflow equivalents in Azure Logic Apps. |
   | **Artifact dispositions** | The artifacts that require conversion and their upload destinations. |
   | **Migration gaps** | The features or components without direct equivalents in Azure Logic Apps (Standard) and the recommended workarounds. |
   | **Integration patterns** | The detected patterns for the integration flow. |
   | **Summary** | A high-level overview about the proposed workflow. |
   | **Effort estimates** | The estimated complexity and effort for each flow. |
   | **Task plans** | The step-by-step conversion tasks for the next stage. |

   The following example shows a sample generated migration plan:

   :::image type="content" source="media/migration-agent-quickstart/planning-stage-main.png" alt-text="Screenshot that shows the Planning stage with the migration plan for a logical group flow and action mappings.":::

1. Review each plan carefully and make any necessary updates.

   > [!TIP]
   >
   > To edit the generated plans, ask questions about specific mappings, or request alternative approaches, interact with the `@migration-planner` agent by using Copilot chat.

1. When you're ready, continue to the Conversion stage by selecting **Home Page** or returning to the **Home** tab.

## Migration stage 3: Conversion

When you're satisfied with your migration plan, start the Conversion stage to generate conversion tasks that transform source artifacts into Standard workflows for Azure Logic Apps.

1. On the **Home** tab, for your logical flow, select **Create Conversion Tasks**.

   :::image type="content" source="media/migration-agent-quickstart/create-conversion-tasks-stage.png" alt-text="Screenshot that shows the Conversion stage for generating conversion tasks.":::

   The `@migration-converter` agent creates conversion tasks, which vary based on your specific logical flow group. The conversion process produces complete, ready-to-run Standard workflow definitions. The `@migration-converter` agent uses the `no-stubs-code-generation` skill to make sure all generated code is fully functional.
   
   The following example shows sample task plans for a logical flow group named `Method Call Processing`:

   1. **Scaffold Logic Apps Project**
   
      Creates the Standard logic app project structure with the required folder hierarchy and files.

   1. **Convert Input Schema**

      Migrates the *InputSchema.xsd* file from BizTalk format, which is UTF-16 with BizTalk annotations, to standard XSD, which is UTF-8 without BizTalk annotations.

   1. **Convert Output Schema**

      Migrates the *OutputSchema.xsd* file from BizTalk format, which is UTF-16 with BizTalk annotations, to standard XSD, which is UTF-8 without BizTalk annotations.

   1. **Generate \<*connector-name*\> Connections**
   
      Creates or updates the *connections.json* file that contains the configurations for each required connection.

   1. **Generate \<*workflow-name*\> Workflow**

      Creates the *workflow.json* file that contains the Standard workflow definition in Azure Logic Apps for the logical flow group.

   1. **Generate Local Functions (\<*function-names*\>)**

      Creates .NET 8 local functions for custom logic in the source code.

   1. **Validate Runtime (func start)**

      Validates the logic app project by running `func start` to confirm that all functions and workflows are ready.

   1. **E2E Testing (Happy Path & Error Path)**

      Runs end-to-end tests for the happy path, error path, and field-level validation.

   1. **Black Box Tests (Optional)**

      Runs tests that use external test data that you provide.

   1. **Cloud Deployment & Testing (Optional)**

      Deploys to Azure and runs cloud E2E tests.

1. To run each conversion task, select **Execute**, and then stop before **Cloud Deployment & Testing**.

   The following example shows sample generated conversion tasks for the `Method Call Processing` logical flow:

   :::image type="content" source="media/migration-agent-quickstart/conversion-stage-main.png" alt-text="Screenshot that shows the Conversion stage with generated conversion tasks that create Standard workflow files for Azure Logic Apps.":::

1. When you finish, continue to the Validation stage by selecting **Home Page** or returning to the **Home** tab.

## Migration stage 4: Validation

After the migration agent converts your source artifacts to Standard workflows, test the generated workflows against your source specifications. The `@migration-converter` agent provides runtime validation and testing guidance. 
The Validation stage lets you bring your own test cases and specifications. Your goal is to confirm that your converted workflow performs as expected.

For this stage, complete the following tasks:

1. On the **Home** tab, for your logical flow, select **Open in Visual Studio Code**.

1. In your migration folder, go to the **out** directory, and select the generated solution folder, for example:

   :::image type="content" source="media/migration-agent-quickstart/validation-stage-generated-output.png" alt-text="Screenshot that shows the local path for where to find the generated code and solution.":::

1. Validate that the generated solution and code are complete. Make sure that no stubs or placeholder code remain.

1. Locally run the generated workflows by using the Azure Functions runtime and the Docker Desktop.

   For example, if you didn't already, run the **Black Box Tests (Optional)** task with external test data that you provide:

   :::image type="content" source="media/migration-agent-quickstart/validation-stage-blackbox.png" alt-text="Screenshot that shows optional Black Box Tests.":::

1. Compare the behavior of the generated workflows against the original integration flows.

1. Identify and fix any discrepancies.

## Migration stage 5: Deployment

This last stage deploys your migrated solution to Azure Logic Apps.

1. Before you start, view the extension's settings. From the **File** menu, go to **Preferences** > **Settings** > **Extensions** > **Logic Apps Migration Agent**.

1. Update the following deployment setting values as appropriate:

   | Setting name | Description | Default | Action |
   |--------------|-------------|---------|--------|
   | **Location** | The Azure region for provisioning resources. | `eastus` | Change this value to the region you want. |
   | **Resource Group** | The Azure resource group for provisioning and testing. | `integration-migration-tool-test-rg` | Change this value to the resource group name you want. |
   | **Subscription ID** | The Azure subscription ID for deployment. | (empty) | Enter the GUID for your Azure subscription. |
   | **Deployment Model** | The target deployment model for Azure Logic Apps (Standard). | `workflow-service-plan` | If appropriate, change this value to `hybrid`. |

1. To start deployment, run the **Cloud Deployment & Testing** task by selecting **Execute**:

   :::image type="content" source="media/migration-agent-quickstart/validation-stage-main.png" alt-text="Screenshot that shows the end to end testing with deployment in target environment.":::

   The migration agent provisions the required infrastructure and deploys your Standard logic app resource and workflows by using the Azure CLI.

   The following example shows a sample completely migrated solution:

   :::image type="content" source="media/migration-agent-quickstart/deployment-stage-final.png" alt-text="Screenshot that shows the end to end testing with deployment in target environment.":::
   
## Reset the migration

You can restart your migration from the beginning. The following command clears the migration state and lets you start again with the Discovery stage.

1. In Visual Studio Code, open the Command Palette (Keyboard: Ctrl+Shift+P).

1. At the prompt, enter **Azure Logic Apps Migration Agent: Reset Migration**.

## Related content

- []
- [Migration agent stage 1: Discovery](migration-agent-discovery-stage.md)
- [Migration agent stage 2: Planning](migration-agent-planning-stage.md)
- [Migration agent stage 3: Conversion](migration-agent-conversion-stage.md)
- [Migration agent stage 4: Validation](migration-agent-validation-stage.md)
- [Migration agent stage 5: Deployment](migration-agent-deployment-stage.md)
- [Add custom parsers for integration platforms](migration-agent-custom-parsers.md)
