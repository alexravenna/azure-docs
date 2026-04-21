---
title: "Stage 4 - Validation: Test Generated Workflows"
titleSuffix: Logic Apps Migration Agent - Azure
description: Learn how the Validation stage in the Logic Apps Migration Agent tests generated Logic Apps workflows against source specifications.
author: haroldcampos
ms.author: hcampos
ms.reviewer: estfan
ms.topic: overview
ms.date: 04/06/2026
---

# Stage 4 - Validation: Test generated workflows

The Validation stage tests the Logic Apps Standard workflows generated in Stage 3 against the original source specifications. This stage helps you verify that the migrated workflows behave correctly before you deploy them to Azure.

## What happens during Validation

During validation, you run the generated workflows locally and compare their behavior with the original integration flows:

1. **Local runtime setup**: The extension uses the Azure Functions runtime and Docker Desktop to run the generated Logic Apps workflows locally.

1. **Connector resources**: Docker Desktop provides local connector resources needed by the workflows, such as file system watchers, message queues, and database connections.

1. **Behavioral testing**: You test the generated workflows with sample inputs and compare outputs to the expected results from the source platform.

1. **Discrepancy identification**: Any differences between the source and target behavior are flagged for investigation and remediation.

## Prerequisites for local testing

Before you run validation, make sure you have:

| Requirement | Purpose |
|---|---|
| [Docker Desktop](https://docs.docker.com/desktop/setup/install/windows-install/) | Runs local connector resources |
| [Azure Functions Core Tools](https://learn.microsoft.com/azure/azure-functions/functions-run-local) | Hosts the Logic Apps runtime locally |
| Azure Logic Apps (Standard) VS Code extension | Provides the local development experience |

## Running workflows locally

To run a generated workflow locally:

1. Open the generated Logic Apps project folder in VS Code.

1. Make sure Docker Desktop is running.

1. Press **F5** or select **Run** > **Start Debugging** to start the Logic Apps runtime locally.

1. The runtime starts and the workflows become available at local endpoints.

1. Send test requests or trigger the workflows using sample input data.

## Validation checklist

Use the following checklist to verify the generated workflows:

- [ ] All triggers fire correctly with the expected input formats.
- [ ] Action sequences execute in the correct order.
- [ ] Data transformations produce the expected output.
- [ ] Conditional logic branches correctly based on input data.
- [ ] Loop constructs process all items as expected.
- [ ] Error handling scopes catch and handle exceptions appropriately.
- [ ] Connection configurations resolve to the correct endpoints.
- [ ] .NET local functions return expected results.

## Fixing issues

If validation reveals discrepancies:

1. Use the `@migration-converter` agent through Copilot chat to discuss the issue.
1. Describe the expected behavior versus the actual behavior.
1. The agent can suggest fixes or regenerate specific parts of the workflow.

> [!TIP]
> Keep the source platform's test data and expected outputs available during validation so you can make direct comparisons.

## Next steps

- [Stage 5 - Deployment: Deploy to Azure](migrate-logic-apps-migration-agent-stage-deployment.md)
- [Stage 3 - Conversion: Generate Logic Apps workflows](migrate-logic-apps-migration-agent-stage-conversion.md)
