---
title: "Migration Stage 3 - Conversion: Create Workflows"
description: "Learn how the Migration Agent converts source integration artifacts into workflows, connections, and other files for migration to Azure Logic Apps (Standard) during the Conversion stage."
services: azure-logic-apps
ms.suite: integration
author: haroldcampos
ms.author: hcampos
ms.reviewers: estfan, azla
ms.topic: conceptual-article
ai-usage: ai-assisted
ms.update-cycle: 365-days
ms.date: 04/27/2026
# Customer intent: As a developer who works with enterprise integration platforms, such as BizTalk Server, MuleSoft, and others, I want to learn how the Azure Logic Apps (Standard) Migration Agent in Visual Studio Code converts source artifacts into workflows, connections, and other supporting files during the Conversion stage.
---

# Migration to Azure Logic Apps Stage 3 - Conversion: Generate workflows (preview)

[!INCLUDE [logic-apps-sku-standard](../includes/logic-apps-sku-standard.md)]

> [!NOTE]
>
> This preview feature is subject to the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

The migration process for integration projects can stall when complex source artifacts are difficult to transform into deployable resources in Azure Logic Apps (Standard). In the Conversion stage, the Azure Logic Apps Migration Agent in Visual Studio Code solves this problem by running the task plans in your migration plan to create complete artifacts, including ready-to-run Standard workflow definitions, connection configurations, and supporting files. During this stage, review the quality for your conversion outputs and prepare them for validation.

This article describes how the Azure Logic Apps Migration Agent maps source integration artifacts to ready-to-deploy Standard logic app project resources.

## Conversion stage actions

In the Azure Logic Apps Migration Agent, after you complete the **Plan Logic App Design** activity, the **Execute Conversion Tasks** activity becomes available. When you select this activity, the `@migration-converter` GitHub Copilot agent processes each task plan and generates the outputs described in these subsections.

### Project scaffold structure

The `@migration-coverter` agent generates a Standard logic app project. This project contains one Standard workflow definition file per logical flow group, a connections configuration file, a host configuration file, and other supporting files:

```
<project-root>/
├── host.json                    # Host configuration for Standard logic app
├── local.settings.json          # Local development settings
├── connections.json             # Connector configurations
├── <workflow-name>/
│   └── workflow.json            # Workflow definition file per flow group
├── <workflow-name-2>/
│   └── workflow.json            # Workflow definition file per flow group
└── lib/                         
    └── custom/
        └── <function-name>.cs   # .NET local function, if necessary
```

### Workflow definition file

For each logical flow group, the `@migration-coverter` agent generates a `workflow.json` file that contains the following workflow operations:

| Operation | Description |
|-----------|-------------|
| [Trigger](../logic-apps-overview.md/#key-terms) | Each workflow always starts with a single trigger, which is the workflow's entry point. The agent maps this trigger from the receive ports or listeners in the source. |
| [Action](../logic-apps-overview.md/#key-terms) | Each workflow has one or more actions that perform tasks. The agent maps these actions from the orchestration shapes, flow processors, or activities in the source. |
| Conditions or loops | Actions that perform control flow logic, such as **If**, **For each**, and **Until**. The agent translates these actions from decision shapes and loops in the source. |
| Scopes | Actions with `run-after` configurations that you can use to set up error handling. |

### Connections configuration

The `@migration-coverter` agent generates a `connections.json` file, which stores the necessary configurations for connector operations in workflows.

The following table describes the high-level connector groups:

| Connector group | Description and examples |
|-----------------|--------------------------|
| **Built-in** | Connectors with operations that run in the same process as the Azure Logic Apps (Standard) runtime. For example, these connectors include **Request**, **File System**, **HTTP**, **Azure Blob Storage**, **Service Bus**, **SQL Server**, **AS2**, **EDIFACT**, **X12**, and others. |
| **Shared** | Connectors with operations that run in multitenant Azure. For example, these connectors include **Salesforce**, **SAP**, **Office 365 Outlook**, **Power BI**, **SharePoint**, and more. Azure Logic Apps supports [1,400+ shared connectors](/connectors/connector-reference/connector-reference-logicapps-connectors) for Microsoft, Azure, and other platforms in the cloud, on-premises, and hybrid environments. |
| **Custom** | Connectors from other publishers or your organization that you create for custom APIs or other services. For more information, see [Create custom built-in connectors for Standard workflows](../create-custom-built-in-connector-standard.md). |

### 4. .NET local functions

When source platform components don't have a direct Logic Apps connector equivalent, the agent generates .NET local functions. Common scenarios include:

- Custom data transformation logic
- Complex parsing or validation rules
- Calls to on-premises systems via custom protocols
- Business rule evaluation

### 5. Completeness check

The agent applies the `no-stubs-code-generation` skill to verify that all generated code is fully functional. No placeholder code, `TODO` comments, or stub implementations are left in the output.

:::image type="content" source="./media/migrate-logic-apps-migration-agent/conversion-stage.png" alt-text="Screenshot that shows the Conversion stage generating Logic Apps Standard workflow files.":::


## Generated output quality

The conversion process produces complete, deployable artifacts. Each generated file follows these standards:

| Standard | Description |
|---|---|
| **No stubs** | All generated code is complete and functional. |
| **Valid JSON** | All `workflow.json` and `connections.json` files are valid and conform to the Logic Apps schema. |
| **Correct references** | Workflow actions reference the correct connections and parameters. |
| **Error handling** | Workflows include appropriate error handling scopes. |

## Reviewing conversion output

After conversion completes, review the generated files:

1. Open the generated Logic Apps project folder in VS Code.
1. Inspect each `workflow.json` to verify the trigger, actions, and control flow match the source behavior.
1. Check `connections.json` for correct connector configurations.
1. Review any generated .NET local functions for correctness.

> [!TIP]
> Use the `@migration-converter` agent through Copilot chat to ask questions about the generated output, request modifications, or regenerate specific workflows.

## Related content

- [Migration automation from integration platforms to Azure Logic Apps](migration-agent-overview.md)
- [Quickstart: Migrate an integration project using the Azure Logic Apps Migration Agent](migrate-logic-apps-migration-agent-quickstart.md)

## Next steps

> [!div class="nextstepaction"]
> [Migration agent stage 4 - Validation: Test workflows](migration-agent-validation-stage.md)
