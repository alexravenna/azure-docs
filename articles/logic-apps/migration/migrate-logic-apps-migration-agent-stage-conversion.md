---
title: "Stage 3 - Conversion: Generate Logic Apps Workflows"
titleSuffix: Logic Apps Migration Agent - Azure
description: Learn how the Conversion stage in the Logic Apps Migration Agent transforms source artifacts into Azure Logic Apps Standard workflows, connections, and supporting files.
author: haroldcampos
ms.author: hcampos
ms.reviewer: estfan
ms.topic: concept-article
author: haroldcampos
ms.author: hcampos
ms.reviewer: estfan
ms.topic: overview
ms.date: 04/06/2026
---

# Stage 3 - Conversion: Generate Logic Apps workflows

The Conversion stage transforms your source integration artifacts into Azure Logic Apps Standard workflows. The `@migration-converter` Copilot agent executes the task plans generated during Planning to produce ready-to-run workflow definitions, connections, and supporting files.

## What happens during Conversion

The `@migration-converter` agent processes each task plan and generates the following outputs:

:::image type="content" source="./media/migrate-logic-apps-migration-agent/conversion-stage.png" alt-text="Screenshot that shows the Conversion stage generating Logic Apps Standard workflow files.":::

### 1. Project scaffolding

The agent creates the Logic Apps Standard project structure:

```
<project-root>/
├── host.json                    # Logic Apps host configuration
├── local.settings.json          # Local development settings
├── connections.json             # Connector configurations
├── <workflow-name>/
│   └── workflow.json            # Workflow definition
├── <workflow-name-2>/
│   └── workflow.json
└── lib/                         # .NET local functions (if needed)
    └── custom/
        └── <function>.cs
```

### 2. Workflow generation

For each flow group, the agent generates a `workflow.json` file that contains:

- **Triggers**: The entry point for the workflow, mapped from the source platform's receive ports or listeners.
- **Actions**: The processing steps, mapped from source orchestration shapes, flow processors, or activities.
- **Conditions and loops**: Control flow logic translated from the source platform's decision shapes and loops.
- **Error handling**: Scopes with run-after configurations for exception handling.

### 3. Connection configuration

The agent generates `connections.json` with the connector configurations needed by the workflows. Connections include:

- **Built-in connectors**: File, HTTP, Service Bus, SQL, and other connectors that run in-process with the Logic Apps runtime.
- **Managed connectors**: Azure-hosted connectors for services such as Salesforce, SAP, and Dynamics 365.
- **Custom connectors**: Configurations for custom APIs and services.

### 4. .NET local functions

When source platform components don't have a direct Logic Apps connector equivalent, the agent generates .NET local functions. Common scenarios include:

- Custom data transformation logic
- Complex parsing or validation rules
- Calls to on-premises systems via custom protocols
- Business rule evaluation

### 5. Completeness check

The agent applies the `no-stubs-code-generation` skill to verify that all generated code is fully functional. No placeholder code, `TODO` comments, or stub implementations are left in the output.

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

## Next steps

- [Stage 4 - Validation: Test generated workflows](migrate-logic-apps-migration-agent-stage-validation.md)
- [Stage 2 - Planning: Create the migration roadmap](migrate-logic-apps-migration-agent-stage-planning.md)
