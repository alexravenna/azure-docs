---
title: "Migration Stage 1 - Discovery: Catalog Artifacts"
description: Learn how the Migration Agent scans, catalogs, and analyzes source artifacts during Discovery stage for migration to Azure Logic Apps (Standard).
services: azure-logic-apps
ms.suite: integration
author: haroldcampos
ms.author: hcampos
ms.reviewers: estfan, azla
ms.topic: conceptual-article
ai-usage: ai-assisted
ms.update-cycle: 365-days
ms.date: 04/27/2026
# Customer intent: As a developer who works with enterprise integration platforms, such as BizTalk Server, MuleSoft, and others, I want to learn how the Azure Logic Apps (Standard) Migration Agent in Visual Studio Code scans my source integration projects to find, catalog, and analyze integration artifacts during the Discovery stage.
---

# Migration Agent Stage 1 - Discovery: Catalog integration artifacts (preview)

[!INCLUDE [logic-apps-sku-standard](../includes/logic-apps-sku-standard.md)]

> [!NOTE]
>
> This preview feature is subject to the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

When you start the migration process, you need to first inventory and understand the source artifacts in your integration projects. In Visual Studio Code, the Azure Logic Apps Migration Agent workflow begins with the Discovery stage by scanning your source project files, detecting the source platform, and cataloging the artifacts and dependencies. The agent then identifies the flow groups, architecture, and migration gaps so you can plan the next stage with better information.

## Discovery stage actions and events

In Visual Studo Code, after you open the Azure Logic Apps Migration Agent on the Activity Bar, and select the source folder, the migration agent automatically completes the following steps by using the `@migration-analyser` GitHub Copilot agent:

| Step | Action | Description |
|------|--------|-------------|
| 1 | **Detect platform** | Examines file patterns to identify the source platform. <br><br>For example, `.btproj` and `.odx` files indicate a BizTalk Server project, while `mule-*.xml` files indicate a MuleSoft Anypoint project. <br><br>For more information, see: <br>- [BizTalk artifact support](migration-agent-overview.md#biztalk-support) <br>- [MuleSoft artifact support](migration-agent-overview.md#mulesoft-support) |
| 2 | **Scan files** | Scans source files and extracts metadata into Intermediate Representation (IR) format by using a built-in parser for the detected platform. |
| 3 | **Catalog artifacts** | Inventories discovered artifacts, including the following items: <br><br>- Orchestrations and workflows <br>- Schemas (XSD, JSON) <br>- Maps and transformations <br>- Pipelines <br>- Send ports and receive ports <br>- Bindings and endpoint configurations |
| 4 | **Build dependency graph** | Generates a dependency graph that shows how artifacts relate to each other. For example, the graph shows which orchestrations reference which schemas and maps. |

## Discovery stage analysis and results

After the migrate agent completes the initial scan, the agent performs a deeper, AI-powered analysis by using the `@migration-analyser` GitHub Copilot agent:

| Action | Description |
|--------|-------------|
| **Detect flow groups** | Groups related artifacts into logical flow groups, which are sets of artifacts that work together to implement a business process. |
| **Visualize architecture** | Generates interactive Mermaid diagrams that show the overall system architecture. |
| **Map message flows** | Traces message flows starting with trigger event, through processing, and to completion for each flow group. |
| **Analyze dependencies** | Identifies missing or unresolved dependencies that might affect migration. |
| **Identify gaps** | Reports source platform features without direct equivalents in Azure Logic Apps (Standard) and recommended workarounds. |
| **Detect patterns** | Identifies common integration patterns such as publish-subscribe, request-reply, scatter-gather, and batch processing. |

For more information, see [Migration stage 1: Discovery](migration-agent-quickstart.md#migration-stage-1-discovery).

## Source design analysis

When you start the source design analysis for a logical flow group, the migration agent generates and opens a flow visualization, for example:

:::image type="content" source="media/migration-agent-discovery-stage/discovery-stage-analysis.png" alt-text="Screenshot that shows the flow visualizer with the results from the Discovery stage.":::

You can switch between the interactive tabs to review your integration architecture. To learn more about this architecture, you can use the GitHub Copilot chat window to ask the `@migration-analyser` agent questions about the detected architecture, request corrections, and regenerate the analsysis.

For more information, see:

- [Flow visualization for integration architecture](migration-agent-overview.md#flow-visualization-for-integration-architecture)
- [Discovery stage - Step 3: Analyze source design](migration-agent-quickstart.md#step-3-analyze-source-design)

## Related content

- [Migration agent stage 2: Planning](migration-agent-planning-stage.md)
- [Migration automation from integration platforms to Azure Logic Apps](migration-agent-overview.md)
