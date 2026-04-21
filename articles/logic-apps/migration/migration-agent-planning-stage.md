---
title: "Stage 2 - Planning: Create the Migration Roadmap"
titleSuffix: Logic Apps Migration Agent - Azure
description: Learn how the Planning stage in the Logic Apps Migration Agent analyzes complexity, maps source components to Logic Apps equivalents, and generates per-flow migration plans.
author: haroldcampos
ms.author: hcampos
ms.reviewer: estfan
ms.topic: overview
ms.date: 04/06/2026
---

# Stage 2 - Planning: Create the migration roadmap

The Planning stage takes the artifacts discovered in Stage 1 and creates a detailed migration roadmap. The `@migration-planner` Copilot agent analyzes each flow group and generates per-flow migration plans with action mappings, gap analysis, and effort estimates.

## What happens during Planning

The `@migration-planner` agent processes each flow group and generates:

1. **Action mappings**: One-to-one mappings from every source platform component to its Azure Logic Apps Standard equivalent. For example, a BizTalk FILE receive port maps to a Logic Apps file trigger, and a BizTalk HTTP send port maps to a Logic Apps HTTP action.

1. **Gap analysis**: Identifies components that don't have a direct Logic Apps equivalent and recommends workarounds. For example, a BizTalk custom pipeline component might require a .NET local function in Logic Apps.

1. **Effort estimates**: Provides estimated complexity (low, medium, high) and effort for each flow based on the number of actions, gaps, and dependencies.

1. **Task plans**: Creates step-by-step conversion tasks that the `@migration-converter` agent executes in Stage 3.

## Migration plan structure

Each flow group gets its own migration plan that includes the following sections:

:::image type="content" source="./media/migrate-logic-apps-migration-agent/planning-stage.png" alt-text="Screenshot that shows the Planning stage with per-flow migration plans and action mappings.":::

### Action mappings

Action mappings describe how each source component translates to Logic Apps:

| Source component | Logic Apps equivalent | Mapping type | Notes |
|---|---|---|---|
| Receive port (FILE) | When a file is added or modified (trigger) | Native | Built-in connector |
| Send port (HTTP) | HTTP action | Native | Built-in connector |
| Orchestration shape (Transform) | Transform XML action | Native | Built-in action |
| Custom pipeline component | Azure Function / .NET local function | Custom | Requires code migration |

### Gap resolution

For each identified gap, the plan includes:

- **Gap description**: What the source component does and why there's no direct equivalent.
- **Recommended resolution**: The suggested approach, such as using a .NET local function, Azure Function, or custom connector.
- **Effort impact**: How the gap affects the overall migration effort estimate.

### Task plans

Task plans are the step-by-step instructions that drive Stage 3 (Conversion). Each task specifies:

- The artifacts to convert
- The target Logic Apps workflow structure
- The connections and configurations to generate
- Any custom code that needs to be written

## Reviewing and adjusting plans

You can interact with the `@migration-planner` agent through Copilot chat to:

- Ask questions about specific mappings
- Request alternative approaches for gap resolutions
- Adjust effort estimates
- Request plan modifications before proceeding to conversion

> [!TIP]
> Review each migration plan carefully before proceeding to Stage 3. The quality of the conversion output depends on the accuracy of the plans.

## Next steps

- [Stage 3 - Conversion: Generate Logic Apps workflows](migrate-logic-apps-migration-agent-stage-conversion.md)
- [Stage 1 - Discovery: Catalog integration artifacts](migrate-logic-apps-migration-agent-stage-discovery.md)
