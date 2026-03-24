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

A mock run lets you simulate the execution of a storage task assignment without actually performing any operations on your blobs. When you create a mock run, Azure Storage Actions scans and evaluates your blobs against the task conditions exactly as it would during a real run, but no operations are executed. Instead, a detailed report is generated showing which blobs matched the conditions and what operations _would have been_ performed.

Mock runs are useful when you want to:

*   **Preview the impact** of a task before running it at scale, especially when operations are irreversible (such as delete or immutability policy).
*   **Validate conditions** on the full set of blobs in your account, not just a small preview sample.
*   **Generate audit-ready reports** showing which blobs would be affected, without making any changes.
*   **Estimate cost** by understanding how many blobs would be targeted and how many operations would be invoked.

> [!NOTE]  
> A mock run scans and evaluates all blobs in scope, just like a real run. The only difference is that no operations are performed on the blobs. Because no operations are executed, mock runs are typically faster than real runs.

### Prerequisites

Before you create a mock run, make sure you have the following:

*   A storage task with at least one condition and one operation defined. See [Create a storage task](storage-task-quickstart-portal).
*   The managed identity associated with the storage task must have the appropriate role (such as **Storage Blob Data Reader** or **Storage Blob Data Owner**) on the target storage account. Although no operations are performed, the identity still needs read access to scan and evaluate blobs.
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
8.  In the **Report** section, select the **Report export container** where the mock run report will be stored. Optionally, set a report prefix path and report format (CSV or Parquet).
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

After a mock run is enabled, you can monitor its progress the same way you monitor a real task run.

> [!NOTE]  
> Only one run (mock or real) can execute on a storage account at a time. If a real run or another mock run is already in progress, the new mock run is queued until the current run completes.

### View mock run reports

When a mock run finishes, a detailed report is written to the report export container you specified during assignment creation. The report lists every blob that was scanned, whether it matched the conditions, and what operation _would have been_ performed.

**Report columns:**

| Column | Description |
| --- | --- |
| **Container** | The container where the blob resides. |
| **Blob** | The name of the blob. |
| **Operation to be performed** | The simulated operation, prefixed with `(mock)` — for example, `(mock) DeleteBlob` or `(mock) SetBlobImmutability`. |
| **Matched condition block** | Which condition block the blob matched (for example, `IF` or `ELSE`). |

**Example CSV output:**

| Container | Blob | Operation to be performed | Matched condition block |
| --- | --- | --- | --- |
| testContainer1 | output1.log | (mock) DeleteBlob | IF |
| testContainer2 | output2.log | (mock) DeleteBlob | IF |
| testContainer1 | financials1.csv | (mock) SetBlobImmutability | ELSE |
| testContainer2 | financials2.csv | (mock) SetBlobImmutability | ELSE |

A **summary JSON** file is also generated alongside the report, containing aggregate metrics:

```
{
  "completionTime": "2024-10-21T17:46:59",
  "destination": "taskoutput",
  "endpoint": "https://contoso1storage1.blob.core.windows.net",
  "fileFormat": "csv",
  "fileSchema": [
    "Container",
    "Blob",
    "Operation to be performed",
    "Result",
    "Matched condition block"
  ],
  "files": [
    "<link to the reporting file>"
  ],
  "objectsListed": 1100,
  "objectsToBeOperated": 240,
  "operationType": "BlobOperation",
  "runId": "mockrun-assignment-2024-10-21T17:30:13.9121342Z",
  "startTime": "2024-10-21T17:37:12",
  "status": "succeeded"
}
```

### Transition from mock run to real run

After reviewing the mock run report and confirming the results are as expected, you can transition the assignment to a real run:

1.  Navigate to the assignment in the Azure portal.
2.  Edit the assignment and change the trigger type from **Mock run** to **Run once** or **Recurring**.
3.  Save the updated assignment.

> [!IMPORTANT]  
> A completed mock run cannot be restarted. To run another mock simulation, you must create a new assignment or duplicate the existing one.

### Pricing

Mock runs are billed similarly to real task assignment runs, with one key difference: **no charge is applied for the operations meter**, since no operations are actually performed on blobs.

| Billing meter | Applies to mock runs? |
| --- | --- |
| Task execution instance (per run) | ✅ Yes |
| Objects targeted (per million objects scanned) | ✅ Yes |
| Operations performed (per million operations) | ❌ No (always $0) |

Standard Blob Storage API costs for listing and reading blob properties during the scan still apply.

> [!TIP]  
> Because mock runs exclude the operations meter charge, they are significantly cheaper than real runs — making them a cost-effective way to validate your task configuration before committing to a full execution.

---