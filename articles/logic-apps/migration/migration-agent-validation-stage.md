---
title: "Migration Stage 4 - Validation: Test Workflows"
description: "Learn how the Migration Agent tests generated workflows in Azure Logic Apps (Standard) against source behavior during the Validation stage."
services: azure-logic-apps
ms.suite: integration
author: haroldcampos
ms.author: hcampos
ms.reviewers: estfan, azla
ms.topic: conceptual-article
ai-usage: ai-assisted
ms.update-cycle: 365-days
ms.date: 04/27/2026
# Customer intent: As a developer who works with enterprise integration platforms, such as BizTalk Server, MuleSoft, and others, I want to learn how the Azure Logic Apps (Standard) Migration Agent in Visual Studio Code verifies the generated workflows, connections, and other supporting files against the source behavior during the Validation stage.
---

# Migration to Azure Logic Apps Stage 4 - Validation: Test workflows (preview)

[!INCLUDE [logic-apps-sku-standard](../includes/logic-apps-sku-standard.md)]

> [!NOTE]
>
> This preview feature is subject to the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

After you generante the Standard logic app project, workflows, and other artifacts for migration to Azure Logic Apps, you need to validate the workflow behavior against the source system's behavior. The validation process can be difficult because behavior differences between source and target systems are easy to miss. For the Validation stage, confirm that the migrated workflows behave correctly before you deploy them to Azure. Confirm that workflow triggers, actions, transformations, and connections work as expected.

This article describes the general process for testing generated Standard workflows against the source behavior, compare the results, and identify any problems or gaps you need to resolve.

## Validation actions

Before you start, make sure that you already inspected the generated project and its artifacts for completeness and quality. For more information, see []().

## Review the generated project and artifacts

inspect the generated project artifacts for completeness and 
to review the generated project files for quality, 

After the agent finishes the conversion, review the generated files by following these steps:

1. Check that the generated solution and code are complete. Make sure that no stubs or placeholder code remain.

1. Inspect each `workflow.json` to verify the trigger, actions, and control flow match the source behavior.

1. Check `connections.json` for correct connector configurations.

1. Review any generated .NET local functions for correctness.

1. Locally run the generated workflows by using the Azure Functions runtime and the Docker Desktop.

The Validation stage tests the Logic Apps Standard workflows generated in Stage 3 against the original source specifications. 


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

## Related content

- [Migration automation from integration platforms to Azure Logic Apps](migration-agent-overview.md)
- [Quickstart: Migrate an integration project using the Azure Logic Apps Migration Agent](migrate-logic-apps-migration-agent-quickstart.md)

## Next steps

> [!div class="nextstepaction"]
> [Migration agent stage 5 - Deployment: Deploy to Azure](migration-agent-validation-deployment.md)
