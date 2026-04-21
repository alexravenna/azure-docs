---
title: Add a Custom Parser for a New Platform
titleSuffix: Logic Apps Migration Agent - Azure
description: Learn how to extend the Logic Apps Migration Agent by adding a custom parser to support a new integration platform such as TIBCO, IBM IIB, or Workato.
author: haroldcampos
ms.author: hcampos
ms.reviewer: estfan
ms.topic: overview
ms.date: 04/06/2026
---

# Add a custom parser for a new platform

This quickstart shows you how to extend the Logic Apps Migration Agent by adding support for a new integration platform. The extension uses a registry-based parser architecture with built-in and external parser support, so you can add platforms such as TIBCO BusinessWorks, IBM IIB/ACE, Dell Boomi, or Workato.

There are two approaches for adding platform support:

| Approach | Recommended | Description |
|---|:-:|---|
| **Built-in parser** (contribute to the repo) | Yes | Add a parser and skills directly to the project. Full integration with all five migration stages. |
| **External parser extension** | | Create a separate VS Code extension that registers parsers via the plugin API. Covers the Discovery stage only. |

Built-in parsers are the recommended approach because they ship with the extension, use the same CI/CD pipeline, and have access to all internal APIs.

## Prerequisites

Before you begin, make sure you have the following resources:

- [Node.js](https://nodejs.org/) 18 or later
- VS Code 1.85.0 or later
- The Logic Apps Migration Agent extension installed
- Familiarity with TypeScript and the VS Code extension API
- A source integration project for the platform you want to support

## Parser architecture

All parsers transform source platform artifacts into a common **Intermediate Representation (IR)** format. The IR is a JSON document that describes integration artifacts in a platform-neutral way. The extension uses the IR throughout the planning, conversion, and validation stages.The parser registry supports both built-in and external parsers:

Built-in parsers:

1. BizTalk (.btproj, .odx)
1. BizTalk (.btm, .xsd)
1. BizTalk (.btp)   
1. MuleSoft (stub)

External parser plugins:

1. Partner Platform Parsers
1. Community Parsers

:::image type="content" source="./media/migrate-logic-apps-migration-agent/parser-architecture.png" alt-text="Diagram that shows how built-in and external parsers feed into the common IR document format used by the migration stages.":::


## Step 1: Add a built-in parser

Create a new parser module under `src/parsers/<your-platform>/`:

```
src/parsers/
├── biztalk/              # Reference implementation
│   ├── index.ts
│   ├── types.ts
│   ├── BizTalkProjectParser.ts
│   ├── BizTalkOrchestrationParser.ts
│   └── ...
├── <your-platform>/      # Your new parser
│   ├── index.ts
│   ├── types.ts
│   └── YourPlatformParser.ts
```

Each parser must implement the `IParser` interface:

```typescript
import { IParser, ParserCapabilities, ParseResult, ParseOptions } from '../types';
import { IRDocument, createEmptyIRDocument } from '../../ir/types';

export class YourPlatformParser implements IParser {
    get capabilities(): ParserCapabilities {
        return {
            platform: 'your-platform',
            fileExtensions: ['.yourext'],
            fileTypes: ['flow'],
            supportsFolder: false,
            description: 'Parses YourPlatform integration flows',
        };
    }

    canParse(filePath: string): boolean {
        return filePath.endsWith('.yourext');
    }

    async parse(
        filePath: string,
        options?: ParseOptions
    ): Promise<ParseResult> {
        const ir = createEmptyIRDocument();
        // Parse the source file and populate the IR document.
        // See docs/IRSchema.md for the complete schema.
        return { ir, stats: { /* parsing statistics */ } };
    }
}
```

Register your parser in `src/parsers/index.ts`:

```typescript
import { YourPlatformParser } from './your-platform';

export function initializeParsers(): void {
    // ... existing parsers ...
    defaultParserRegistry.register(new YourPlatformParser());
}
```

> [!TIP]
> Use the BizTalk parser implementation in `src/parsers/biztalk/` as a fully working reference.

## Step 2: Add platform-specific skills

Skills are Markdown files that provide AI instructions for each stage of the migration. They tell the Copilot agents how to analyze, plan, and convert artifacts for your specific platform.

Skills are organized under `resources/skills/`, with platform-specific variants:

```
resources/skills/
├── detect-logical-groups/
│   ├── biztalk/SKILL.md
│   ├── mulesoft/SKILL.md
│   └── <your-platform>/SKILL.md
├── source-to-logic-apps-mapping/
│   ├── biztalk/SKILL.md
│   ├── mulesoft/SKILL.md
│   └── <your-platform>/SKILL.md
└── ... (13 skills total)
```

Each `SKILL.md` uses YAML frontmatter followed by Markdown content:

```markdown
---
name: source-to-logic-apps-mapping
description: >-
  Component mapping of YourPlatform components
  to their Azure Logic Apps Standard equivalents.
---

# YourPlatform to Azure Logic Apps Standard component mapping

## Adapter mappings

| YourPlatform Component | Logic Apps Equivalent | Native? | Notes |
|---|---|---|---|
| HTTP Listener | HTTP Trigger | Yes | Native built-in |
| Database Connector | SQL Server connector | Yes | Native built-in |
```

### Required skills

You should create a `<your-platform>/SKILL.md` variant for each of the following 13 skills:

| Skill | Purpose | Agent |
|---|---|---|
| `detect-logical-groups` | Rules for grouping artifacts into logical flow groups | `@migration-analyser` |
| `analyse-source-design` | Rules for analyzing source architecture and generating visualizations | `@migration-analyser` |
| `dependency-and-decompilation-analysis` | Rules for identifying missing dependencies | `@migration-analyser` |
| `source-to-logic-apps-mapping` | Component-by-component mapping from source to Logic Apps | All agents |
| `logic-apps-planning-rules` | Rules for generating migration plans | `@migration-planner` |
| `conversion-task-plan-rules` | Rules for creating conversion task plans | `@migration-converter` |
| `scaffold-logic-apps-project` | Rules for scaffolding the Logic Apps project structure | `@migration-converter` |
| `workflow-json-generation-rules` | Rules for generating `workflow.json` files | `@migration-converter` |
| `connections-json-generation-rules` | Rules for generating `connections.json` | `@migration-converter` |
| `dotnet-local-functions-logic-apps` | Rules for generating .NET local functions | `@migration-converter` |
| `no-stubs-code-generation` | Rules ensuring generated code is complete | `@migration-converter` |
| `runtime-validation-and-testing` | Rules for runtime validation and testing | `@migration-converter` |
| `cloud-deployment-and-testing` | Rules for cloud deployment and testing | `@migration-converter` |

> [!TIP]
> Copy an existing platform's skills (for example, `biztalk/SKILL.md`) as a starting point and adapt the content for your platform.

## Step 3: Register the platform

Add your platform to the supported platforms list in `src/types/platforms.ts`:

```typescript
export type SourcePlatform = 'biztalk' | 'mulesoft' | 'your-platform';

export const SUPPORTED_PLATFORMS: PlatformInfo[] = [
    // ... existing platforms ...
    {
        id: 'your-platform',
        label: 'YourPlatform',
        description: 'YourPlatform version X',
        icon: '$(server)',
        filePatterns: ['.yourext', '.yourconfig'],
    },
];
```

Also add detection logic in `src/stages/discovery/PlatformDetector.ts` and file patterns in `src/stages/discovery/SourceFolderService.ts`.

## Step 4: Add IR examples (optional)

Add a `docs/IRExamples_YourPlatform.md` file that documents how your platform's artifacts map to the IR schema. The following existing examples serve as templates:

- `docs/IRExamples_BizTalk.md` — BizTalk reference
- `docs/IRExamples_MuleSoft.md` — MuleSoft reference
- `docs/IRExamples_Boomi.md` — Dell Boomi example
- `docs/IRExamples_IBMIIB.md` — IBM IIB/ACE example
- `docs/IRExamples_TIBCO.md` — TIBCO BusinessWorks example
- `docs/IRExamples_Workato.md` — Workato example

## Alternative: External parser extension

If you prefer not to contribute directly to the repository, you can create a separate VS Code extension that registers parsers via the plugin API:

```typescript
import * as vscode from 'vscode';

export async function activate(
    context: vscode.ExtensionContext
) {
    const assistant = vscode.extensions.getExtension(
        'microsoft.logicapps-migration-assistant'
    );

    if (assistant) {
        const api = await assistant.activate();
        api.registerParser(new MyPlatformParser(), {
            priority: 10,
        });
    }
}
```

> [!NOTE]
> External parser extensions only cover the **Discovery** stage (parsing source files). Skills, platform detection, and AI-powered planning and conversion require contributing to the main repository directly.

### Parser plugin API

| Method or property | Description |
|---|---|
| `version` | Extension version (read-only) |
| `registerParser(parser, options?)` | Register a parser with the registry |
| `unregisterParser(id)` | Remove a registered parser |
| `getParserRegistry()` | Access the parser registry directly |
| `hasParser(id)` | Check if a parser is registered |
| `getExternalParsers()` | Get information about registered external parsers |

## Next steps

- [Logic Apps Migration Agent overview](migrate-logic-apps-migration-agent-overview.md)
- [Quickstart: Migrate an integration project using the Logic Apps Migration Agent](migrate-logic-apps-migration-agent-quickstart.md)
