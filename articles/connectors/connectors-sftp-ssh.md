---
title: Connect to SFTP servers from Workflows
description: Manage files on SFTP servers over Secure Shell (SSH) protocol from workflows in Azure Logic Apps.
services: azure-logic-apps
ms.suite: integration
ms.reviewers: estfan, azla
ms.topic: how-to
ms.update-cycle: 180-days
ai-usage: ai-assisted
ms.date: 05/11/2026
#Customer intent: As an automation and integration developer who works with Azure Logic Apps, I want to connect and manage files on SFTP servers from my workflows by using Secure Shell (SSH) protocol.
---

# Connect to files on SFTP servers over Secure Shell (SSH) from workflows in Azure Logic Apps

[!INCLUDE [logic-apps-sku-consumption-standard](../../includes/logic-apps-sku-consumption-standard.md)]

When your workflow needs to perform automated, secure file management on servers that use the Secure Shell (SSH) File Transfer Protocol (SFTP), use the **SFTP-SSH** or **SFTP** connector operations in the workflows that you create with Azure Logic Apps.

SFTP is a network protocol that provides file access, file transfer, and file management over any reliable data stream. You can then monitor, transfer, and manage files on your SFTP server — without writing custom code or managing infrastructure. Otherwise, trying to manually manage these file operations can be time-consuming, error-prone, and hard to scale.

For example, your workflow can complete the following tasks:

- Monitor and process incoming data files.
- Create and manage folders and files. 
- Get file content and metadata. Extract archives.
- Distribute reports.
- Synchronize content across environments.

This guide shows how to access your SFTP server from a workflow in Azure Logic Apps.

For more information, see:

- [SSH File Transfer Protocol (SFTP)](https://www.ssh.com/ssh/sftp/)
- [Secure Shell (SSH)](https://www.ssh.com/ssh/protocol/)

## Connector technical reference

The **SFTP-SSH** connector has different versions, based on [logic app type and host environment](../logic-apps/logic-apps-overview.md#resource-environment-differences).

Consumption and Standard workflows can use the **SFTP-SSH** *managed* connector, which shares compute with other resources in multitenant Azure. Standard workflows can also use the runtime-native or *built-in* **SFTP** connector. Both connector versions use the SSH protocol.

| Logic app type (plan) | Environment | Connector version |
|-----------------------|-------------|-------------------|
| **Consumption** | Multitenant Azure Logic Apps | Managed connector, which appears in the connector gallery under the **Shared** filter. <br><br>For more information, see [SFTP-SSH managed connector reference](/connectors/sftpwithssh/). |
| **Standard** | Single-tenant Azure Logic Apps, App Service Environment v3 (Windows plan only), and Hybrid | - Managed connector, which appears in the connector gallery under the **Shared** filter. <br><br>- Built-in connector, which appears in the connector gallery under the **Built in** filter and is [service provider-based](../logic-apps/custom-connector-overview.md#service-provider-interface-implementation). The built-in connector can directly connect to an SFTP server and access Azure virtual networks by using a connection string without an on-premises data gateway. <br><br>For more information, see: <br><br>- [SFTP-SSH managed connector reference](/connectors/sftpwithssh/) <br>- [SFTP built-in connector reference](/azure/logic-apps/connectors/built-in/reference/sftp/) |

Different SFTP connector versions offer different prebuilt operations. You can start a blank workflow with an SFTP-specific trigger, or choose a different trigger, based on your scenario. For example, you can start your workflow with an SFTP trigger that monitors and responds to events on your SFTP server. The trigger provides outputs to use with subsequent actions in your workflow. Various SFTP actions perform different tasks, such as get, create, and manage files on your SFTP server.

## Prerequisites

- An Azure account and subscription. [Get a free Azure account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).

- Information about your SFTP server connection and authentication:

  - Server address
  - Account credentials
  - Access to an SSH private key
  - SSH private key password

  > [!IMPORTANT]
  >
  > When you create your connection and enter your SSH private key in the **SSH private key* parameter, make sure to follow the [steps for providing the complete and correct parameter value](/connectors/sftpwithssh/#authentication-and-permissions). Otherwise, a non-valid key causes the connection to fail.
  
  For more information, see [SFTP-SSH managed connector reference - Authentication and permissions](/connectors/sftpwithssh/#authentication-and-permissions).

- The workflow where you want to access your SFTP server.

  To start your workflow with an SFTP trigger, you need a blank workflow. To use an SFTP action, use any trigger that works best for your scenario. The example in this guide uses the **Recurrence** trigger.

## General limitations

- Before you use the SFTP-SSH managed connector, see [SFTP-SSH managed connector reference - known issues and limitations](/connectors/sftpwithssh/).

- Before you use the SFTP built-in connector, see [SFTP built-in connector reference - known issues and limitations](/azure/logic-apps/connectors/built-in/reference/sftp/).

<a name="known-issues"></a>

## Known issues

[!INCLUDE [Managed connector trigger "Split On" setting issue](../../includes/connectors-managed-trigger-split-on.md)]

## Chunking

Chunking lets an operation handle large files that exceed the default size limits. For more information about the SFTP-SSH managed connector and chunking support, see [SFTP-SSH managed connector reference - Chunking](/connectors/sftpwithssh/#chunking).

## Add an SFTP trigger

Every workflow starts with a trigger. To add a trigger, you need a blank workflow. The following steps show how to add an SFTP trigger to your blank workflow.

<a name="add-managed-sftp-trigger"></a>

### Add a managed SFTP-SSH trigger (Consumption, Standard)

To add and set up a managed or "shared" **SFTP-SSH** connector trigger, follow these steps:

1. In the [Azure portal](https://portal.azure.com), open the logic app resource. In the designer, open the blank workflow.

1. In the designer, follow these [general steps](../logic-apps/add-trigger-action-workflow.md?#add-trigger) to add the shared **SFTP-SSH** trigger.

1. If prompted, provide the necessary [connection information](/connectors/sftpwithssh/#creating-a-connection). When you finish, select **Create new**.

1. On the designer, select the trigger. In the trigger information pane, provide the necessary trigger information.

   For more information, see [SFTP-SSH managed connector triggers reference](/connectors/sftpwithssh/#triggers).

1. When you finish, save your workflow. On the designer toolbar, select **Save**.

<a name="add-built-in-trigger"></a>

## Add a built-in SFTP trigger (Standard only)

To add and set up a built-in **SFTP** connector trigger, follow these steps:

1. In the [Azure portal](https://portal.azure.com), open the logic app resource. In the designer, open the blank workflow.

1. In the designer, follow these [general steps](../logic-apps/add-trigger-action-workflow.md?tabs=standard#add-trigger) to add the built-in **SFTP** trigger.

1. If prompted, provide the necessary [connection information](/azure/logic-apps/connectors/built-in/reference/sftp/#authentication). When you finish, select **Create new**.

1. On the designer, select the trigger. In the trigger information pane, provide the necessary trigger information.

   For more information, see [SFTP built-in connector trigger reference](/azure/logic-apps/connectors/built-in/reference/sftp/#triggers).

1. When you finish, save your workflow. On the designer toolbar, select **Save**.

When you save your workflow, Azure automatically publishes the updates to your deployed logic app that is live in Azure. With only a trigger, your workflow only checks the SFTP server based on your specified schedule. You have to [add an action](#add-sftp-action) that responds to the trigger and does something with the trigger outputs.

For example, the trigger named **When a file is added or modified** starts the workflow when a file is added or changed on an SFTP server. As a subsequent action, you can add a condition that checks whether the file content meets your specified criteria. If the content meets the condition, use the action named **Get file content** to get the file content, and then use another action to put that file content into a different folder on the SFTP server.

<a name="add-sftp-action"></a>

## Add an SFTP action

Before you can use an SFTP action, your workflow must already start with a trigger, which can be any kind that you choose. For example, you can use the generic **Recurrence** built-in trigger to start your workflow on specific schedule.

### [Consumption](#tab/consumption)

1. In the [Azure portal](https://portal.azure.com), open your Consumption logic app with workflow in the designer.

1. In the designer, [follow these general steps to add the SFTP-SSH action that you want](../logic-apps/create-workflow-with-trigger-or-action.md?tabs=consumption#add-action).

1. If prompted, provide the necessary [connection information](/connectors/sftpwithssh/#creating-a-connection). When you're done, select **Create**.

1. After the action information box appears, provide the necessary details for your selected action. For more information, see [SFTP-SSH managed connector actions reference](/connectors/sftpwithssh/#actions).

1. When you're done, save your workflow. On the designer toolbar, select **Save**.

### [Standard](#tab/standard)

<a name="built-in-connector-action"></a>

#### Built-in connector action

1. In the [Azure portal](https://portal.azure.com), open your Standard logic app with workflow in the designer.

1. In the designer, [follow these general steps to add the SFTP-SSH built-in action that you want](../logic-apps/create-workflow-with-trigger-or-action.md?tabs=standard#add-action).

1. If prompted, provide the necessary [connection information](/connectors/sftpwithssh/#creating-a-connection). When you're done, select **Create**.

1. After the action information box appears, provide the necessary details for your selected action. For more information, see [SFTP built-in connector actions reference](/azure/logic-apps/connectors/built-in/reference/sftp/#actions).

1. When you're done, save your workflow. On the designer toolbar, select **Save**.

<a name="managed-connector-action"></a>

#### Managed connector action

1. In the [Azure portal](https://portal.azure.com), open your Standard logic app with workflow in the designer.

1. In the designer, [follow these general steps to add the SFTP-SSH managed action that you want](../logic-apps/create-workflow-with-trigger-or-action.md?tabs=standard#add-action).

1. If prompted, provide the necessary [connection information](/connectors/sftpwithssh/#creating-a-connection). When you're done, select **Create**.

1. After the action information box appears, provide the necessary details for your selected action. For more information, see [SFTP-SSH managed connector actions reference](/connectors/sftpwithssh/#actions).

1. When you're done, save your workflow. On the designer toolbar, select **Save**.

---

For example, the action named **Get file content using path** gets the content from a file on an SFTP server by specifying the file path. You can use the trigger from the previous example and a condition that the file content must meet. If the condition is true, a subsequent action can get the content.

---

## Troubleshooting

For more information, see the following documentation: 

- [SFTP-SSH managed connector reference - Troubleshooting](/connectors/sftpwithssh/#troubleshooting)
- [SFTP built-in connector reference - Troubleshooting](/azure/logic-apps/connectors/built-in/reference/sftp#troubleshooting)

## Next steps

- [Managed connectors for Azure Logic Apps](/connectors/connector-reference/connector-reference-logicapps-connectors)
- [Built-in connectors for Azure Logic Apps](../connectors/built-in.md)
