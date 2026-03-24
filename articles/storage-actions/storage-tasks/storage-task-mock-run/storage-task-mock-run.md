---
# Required metadata
# For more information, see https://learn.microsoft.com/en-us/help/platform/learn-editor-add-metadata
# For valid values of ms.service, ms.prod, and ms.topic, see https://learn.microsoft.com/en-us/help/platform/metadata-taxonomies

title: Create and use a mock run
description: Learn how to create a mock run to simulate a storage task assignment without performing any operations on your blobs.
author: Shashank Kumar Shankar
ms.author:   shashankar # Microsoft alias
ms.service: azure-storage-actions
ms.topic: feature-guide
ms.date: 04/12/2026
ms.subservice: mock run
---

### Introduction

This article shows you how to create a mock run for a storage task assignment. A mock run simulates task execution — scanning and evaluating blobs against your conditions — without performing any operations. For a conceptual overview of mock runs, see [Mock runs for storage task assignments](storage-task-mock-run).

### Prerequisites

Before you create a mock run, ensure you have the following:

*   A storage task with at least one condition and one operation defined. See [Create a storage task](storage-task-quickstart-portal).
*   The managed identity associated with the storage task must have the appropriate role (such as **Storage Blob Data Reader** or **Storage Blob Data Owner**) on the target storage account.
*   If the target storage account has network restrictions, ensure that the **Allow trusted Microsoft services** option is enabled.

### Create a mock run in the Azure portal

#### From a storage task

1.  Navigate to your storage task in the Azure portal.
2.  Under **Storage task management**, select **Assignments**.
3.  Select **\+ Add assignment**. The **Add assignment** pane appears.
4.  In the **Select scope** section:
    *   Select the **Subscription** containing the target storage account.
    *   Provide a **Storage task assignment name** (2–62 characters, letters and numbers only).
    *   Select the target **Storage account**.
5.  In the **Role assignment** section, select the role to assign to the managed identity. Use a role with at least Blob Data Reader permissions to ensure the mock run can scan blobs successfully.
6.  In the **Filter objects** section, configure prefix filters to narrow the scope of blobs to evaluate, or select **Do not filter** to evaluate the entire account.
7.  In the **Trigger details** section:
    *   Select **Mock run** as the run type.
    *   Set the **Start from** date and time.
    *   Optionally configure a **Max duration** for the run.
1. In the **Report** section, select the **Report export container** where the mock run report will be stored.

9.  Select **Add** to create the assignment.
10.  After the assignment appears in the **Assignments** page, select the checkbox next to it and select **Enable** to schedule the mock run.

#### From a storage account

1.  Navigate to the storage account in the Azure portal.
2.  Under **Data management**, select **Storage tasks**.
3.  Select the **Task assignment** tab, then **\+ Create assignment** > **\+ Add assignment**.
4.  Follow steps 4–10 above, but instead of selecting a storage account, select the **Storage task** you want to assign.

> [!TIP]  
> Use the condition preview feature in the **Add assignment** pane to spot-check your conditions on a small sample of blobs before creating the mock run.

### Monitor a mock run

After a mock run is enabled, you can monitor its progress the same way you monitor a real task run. For information about mock run states and what to expect, see [Mock run lifecycle and states](storage-task-mock-run#mock-run-lifecycle-and-states).

### View mock run reports

When a mock run finishes, a detailed report is available in the report export container. For information about report format, columns, and the summary JSON, see [Mock run reports](storage-task-mock-run#mock-run-reports).

To download the report:

1.  Navigate to the assignment in the Azure portal.
2.  Select **Go to mock run report** (or navigate to the report export container directly).
3.  Download the CSV file.

### Transition to a real run

After reviewing the mock run report, you can transition the assignment to a real run:

1.  Navigate to the assignment in the Azure portal.
2.  Edit the assignment and change the trigger type from **Mock run** to **Run once** or **Recurring**.
3.  Save the updated assignment.

> [!IMPORTANT]  
> A completed mock run cannot be restarted. To run another mock simulation, create a new assignment or duplicate the existing one.

### See also

*   [Mock runs for storage task assignments](storage-task-mock-run)
*   [Create and manage a storage task assignment](storage-task-assignment-create)
*   [Analyze storage task runs](storage-task-runs)
