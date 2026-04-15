---
title: "Stage 1 - Discovery: Catalog Integration Artifacts"
titleSuffix: Logic Apps Migration Agent - Azure
description: Learn how the Discovery stage in the Logic Apps Migration Agent scans, catalogs, and analyzes your source integration artifacts.
author: haroldcampos
author: haroldcampos
ms.author: hcampos
ms.reviewer: estfan
ms.topic: overview
ms.date: 04/06/2026
---

# Stage 1 - Discovery: Catalog integration artifacts

The Discovery stage is the starting point of the Logic Apps Migration Agent workflow. In this stage, the extension scans your source project, identifies the integration platform, and builds a complete inventory of all artifacts and their dependencies.

## What happens during Discovery

When you select a source folder, the extension performs the following actions automatically:

1. **Platform detection**: The extension examines file patterns to identify the source platform. For example, `.btproj` and `.odx` files indicate a BizTalk Server project, while `mule-*.xml` files indicate a MuleSoft Anypoint project.

1. **File scanning**: The built-in parser for the detected platform scans all source files and extracts metadata into the Intermediate Representation (IR) format.

1. **Artifact inventory**: The extension catalogs every discovered artifact, including:
   - Orchestrations and workflows
   - Schemas (XSD, JSON)
   - Maps and transformations
   - Pipelines
   - Send ports and receive ports
   - Bindings and endpoint configurations

1. **Dependency graph**: The extension builds a dependency graph that shows how artifacts relate to each other, such as which orchestrations reference which schemas and maps.

## AI-powered analysis

After the initial scan, the `@migration-analyser` Copilot agent performs deeper analysis:

- **Flow group detection**: Groups related artifacts into logical flow groups — sets of artifacts that work together to implement a business process.
- **Architecture visualization**: Generates interactive Mermaid diagrams showing the overall system architecture.
- **Message flow mapping**: Traces message flows from trigger through processing to completion for each flow group.
- **Dependency analysis**: Identifies missing or unresolved dependencies that could affect migration.
- **Gap analysis**: Flags source platform features that don't have a direct equivalent in Azure Logic Apps Standard, with suggested workarounds.
- **Pattern detection**: Identifies common integration patterns such as publish/subscribe, request-reply, scatter-gather, and batch processing.

## Flow Visualizer

The Discovery stage populates the **Flow Visualizer** panel, which provides interactive tabs for reviewing your integration architecture:

:::image type="content" source="./media/migrate-logic-apps-migration-agent/discovery-stage.png" alt-text="Screenshot that shows the Discovery stage in the Logic Apps Migration Agent with the Flow Visualizer panel.":::

| Tab | Description |
|---|---|
| **Architecture Diagram** | System architecture diagram with all artifacts and connections. |
| **Message Flow** | Per-artifact message flow from trigger to completion. |
| **Components** | Component inventory with details such as adapters, endpoints, and pipelines. |
| **Missing Dependencies** | Dependencies that couldn't be resolved during discovery. |
| **Gap Analysis** | Features without a direct Logic Apps equivalent, with suggested resolutions. |
| **Patterns** | Detected integration patterns. |

> [!TIP]
> Select any flow group in the Flow Visualizer and use the Copilot chat integration to ask the `@migration-analyser` agent questions about the detected architecture or request corrections.

## Supported artifact types

### BizTalk Server

| Artifact type | File extension | Description |
|---|---|---|
| Project | `.btproj` | BizTalk project file |
| Orchestration | `.odx` | BizTalk orchestration definition |
| Schema | `.xsd` | XML schema definition |
| Map | `.btm` | BizTalk map (XSLT transformation) |
| Pipeline | `.btp` | BizTalk pipeline definition |
| Bindings | `.xml` | Port bindings and endpoint configuration |

### MuleSoft Anypoint (in progress)

| Artifact type | File pattern | Description |
|---|---|---|
| Flow | `mule-*.xml` | Mule flow definitions |
| Configuration | `pom.xml` | Project dependencies and configuration |

## Next steps

- [Stage 2 - Planning: Create the migration roadmap](migrate-logic-apps-migration-agent-stage-planning.md)
- [Logic Apps Migration Agent overview](migrate-logic-apps-migration-agent-overview.md)
