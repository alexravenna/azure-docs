---
title: Release Notes for Azure File Sync
description: Release notes for Azure File Sync, which lets you centralize your organization's file shares in Azure Files while maintaining fast local access to frequently used data.
services: storage
author: wmgries
ms.service: azure-file-storage
ms.topic: release-notes
ms.date: 07/08/2025
ms.author: wgries
ms.custom:
  - build-2025
# Customer intent: "As a cloud administrator, I want to review the latest release notes for Azure File Sync so that I can stay informed about new features, bug fixes, and support status for proper planning and management of my file storage deployments."
---

# Release notes for Azure File Sync

Azure File Sync enables centralizing your organization's file shares in Azure Files, while keeping the flexibility, performance, and compatibility of a Windows file server. While some users may opt to keep a full copy of their data locally, Azure File Sync additionally has the ability to transform Windows Server into a quick cache of your Azure file share. You can use any protocol that's available on Windows Server to access your data locally, including SMB, NFS, and FTPS. You can have as many caches as you need across the world.

This article provides the release notes for Azure File Sync. It's important to note that major releases of Azure File Sync include service and agent improvements (for example, 18.0.0.0). Minor releases of Azure File Sync are typically for agent improvements (for example, 18.2.0.0).

## Supported versions

The following Azure File Sync agent versions are supported:

| Milestone | Agent version number | Release date | Status |
|----|----------------------|--------------|------------------|
| V21.2 Release - [KB5063825](https://support.microsoft.com/topic/6490aef8-ebae-465d-beba-718c05f8a71f)|21.2.0.0| July 08, 2025| Supported – Flighting |
| V20 Release - [KB5041884](https://support.microsoft.com/topic/b92c9c6f-8232-42d3-a3e0-e6df1fce0f5e)| 20.0.0.0 | February 10, 2025 | Supported |
| V19 Release - [KB5040924](https://support.microsoft.com/topic/e44fc142-8a24-4dea-9bf9-6e884b4b342e)| 19.1.0.0 | September 3, 2024 | Supported |
| V18.2 Release - [KB5023059](https://support.microsoft.com/topic/613d00dc-998b-4885-86b9-73750195baf5)| 18.2.0.0 | July 9, 2024 | Supported |
| V18.1 Release - [KB5023057](https://support.microsoft.com/topic/961af341-40f2-4e95-94c4-f2854add60a5)| 18.1.0.0 | June 11, 2024 | Supported - Security Update |
| V17.3 Release - [KB5039814](https://support.microsoft.com/topic/97bd6ab9-fa4c-42c0-a510-cdb1d23825bf)| 17.3.0.0 | June 11, 2024 |  Not Supported (Security Update) - Agent version expired on June 9, 2025 |
| V18 Release - [KB5023057](https://support.microsoft.com/topic/feb374ad-6256-4eeb-9371-eb85071f756f)| 18.0.0.0 | May 8, 2024 | Supported |


## Unsupported versions

The following Azure File Sync agent versions have expired and are no longer supported:

| Milestone | Agent version number | Release date | Status |
|----|----------------------|--------------|------------------|
| V17 Release | 17.0.0.0 - 17.2.0.0 | N/A | Not Supported - Agent versions expired on June 9, 2025 |
| V16 Release | 16.0.0.0 - 16.2.0.0 | N/A | Not Supported - Agent versions expired on October 7, 2024 |
| V15 Release | 15.0.0.0 - 15.2.0.0 | N/A | Not Supported - Agent versions expired on March 19, 2024 |
| V14 Release | 14.0.0.0 | N/A | Not Supported - Agent versions expired on February 8, 2024 |
| V13 Release | 13.0.0.0 | N/A | Not Supported - Agent versions expired on August 8, 2022 |
| V12 Release | 12.0.0.0 - 12.1.0.0 | N/A | Not Supported - Agent versions expired on May 23, 2022 |
| V11 Release | 11.1.0.0 - 11.3.0.0 | N/A | Not Supported - Agent versions expired on March 28, 2022 |
| V10 Release | 10.0.0.0 - 10.1.0.0 | N/A | Not Supported - Agent versions expired on June 28, 2021 |
| V9 Release | 9.0.0.0 - 9.1.0.0 | N/A | Not Supported - Agent versions expired on February 16, 2021 |
| V8 Release | 8.0.0.0 | N/A | Not Supported - Agent versions expired on January 12, 2021 |
| V7 Release | 7.0.0.0 - 7.2.0.0 | N/A | Not Supported - Agent versions expired on September 1, 2020 |
| V6 Release | 6.0.0.0 - 6.3.0.0 | N/A | Not Supported - Agent versions expired on April 21, 2020 |
| V5 Release | 5.0.2.0 - 5.2.0.0 | N/A | Not Supported - Agent versions expired on March 18, 2020 |
| V4 Release | 4.0.1.0 - 4.3.0.0 | N/A | Not Supported - Agent versions expired on November 6, 2019 |
| V3 Release | 3.1.0.0 - 3.4.0.0 | N/A | Not Supported - Agent versions expired on August 19, 2019 |
| Pre-GA agents | 1.1.0.0 - 3.0.13.0 | N/A | Not Supported - Agent versions expired on October 1, 2018 |

### Azure File Sync agent update policy

[!INCLUDE [storage-sync-files-agent-update-policy](../../../includes/storage-sync-files-agent-update-policy.md)]

## Windows Server 2012 R2 agent support

Windows Server 2012 R2 reached [end of support](/lifecycle/announcements/windows-server-2012-r2-end-of-support) on October 10, 2023. **Azure File Sync will continue to support Windows Server 2012 R2 until the v17.x agent expires on June 9, 2025.** After this date, we will no longer provide bug fixes, security updates, or technical support. 

Although official support for Windows Server 2012 R2 and the Azure File Sync v17 agent will end on June 9, 2025, the v17 agent will continue to function until **January 27, 2026**. After this date, servers with the v17 agent will stop syncing to your Azure file shares.

### Action Required

Perform one of the following options for your Windows Server 2012 R2 servers prior to v17 agent expiration on June 9, 2025:

- Option #1: Perform an [in-place upgrade](/windows-server/get-started/perform-in-place-upgrade) to a [supported operating system version](file-sync-planning.md#operating-system-requirements). Before upgrading, uninstall the Azure File Sync agent and restart the server. ***Don't unregister and re-register the server during the OS upgrade, or it will lead to orphaned tiered files on existing server endpoints.*** Then, perform the in-place upgrade to the new version of Windows Server, such as 2016, 2019, 2022, or 2025. After the upgrade is complete, install the Azure File Sync agent that matches the new operating system version. Once the agent is installed on the upgraded server, the Azure portal should reflect the correct server status within 30 minutes.

- Option #2: Deploy a new Azure File Sync server that's running a [supported operating system version](file-sync-planning.md#operating-system-requirements) to replace your Windows 2012 R2 servers. For guidance, see [Replace an Azure File Sync server](file-sync-replace-server.md).

>[!NOTE]
>Azure File Sync agent v17.3 is the last agent release currently planned for Windows Server 2012 R2. To continue to receive product improvements and bug fixes, upgrade your servers to Windows Server 2016 or later.
## Version 21.2.0.0
The following release notes are for Azure File Sync version 21.2.0.0 (released July 08, 2025). This release contains improvements for the Azure File Sync service and agent. 

### Improvements and issues that are fixed
**Generally Available: Azure File Sync in Italy North**

The expansion into Italy North brings the service closer to organizations in these regions, offering lower latency, better performance, and support for local data residency requirements.  

**Granular Role-Based Access Control (RBAC) for Azure File Sync ​​​​​​**

Azure File Sync now includes two dedicated built-in RBAC roles. [Azure File Sync Administrator](/azure/role-based-access-control/built-in-roles/storage#azure-file-sync-administrator) and [Azure File Sync Reader](/azure/role-based-access-control/built-in-roles/storage#azure-file-sync-reader) are designed to strengthen security and streamline operations for organizations managing file synchronization between on-premises and cloud environments. These roles offer more granular access control than broader roles like Owner or Contributor, enabling organizations to enforce the principle of least privilege more effectively.

**Miscellaneous reliability and telemetry improvements for cloud tiering and sync​​​​​​​**

Fixed an issue that caused sync download and cloud tiering failures with error code 0x80c80362 (ECS_E_ITEM_PATH_COMPONENT_HAS_ TRAILING_DOT) when a file or folder path included a trailing dot. Previously, users had to manually rename the affected item to resume sync. Azure File Sync now correctly handles path components with trailing dots, eliminating the need for manual intervention.

### Evaluation Tool
Before deploying Azure File Sync, you should evaluate whether it's compatible with your system using the Azure File Sync evaluation tool. This tool is an Azure PowerShell cmdlet that checks for potential issues with your file system and dataset, such as unsupported OS version. For installation and usage instructions, see [Evaluation Tool](file-sync-planning.md#evaluation-cmdlet) section in the planning guide.

### Agent installation and server configuration
For more information on how to install and configure the Azure File Sync agent with Windows Server, see [Planning for an Azure File Sync deployment](file-sync-planning.md) and [How to deploy Azure File Sync](file-sync-deployment-guide.md).

- The agent installation requires a restart for servers that have an existing Azure File Sync agent installation if the agent version is older than 18.2.0.0.
- The agent installation package must be installed with elevated (admin) permissions.
- The agent isn't supported on Nano Server deployment option.
- The agent is supported only on Windows Server 2016, Windows Server 2019, Windows Server 2022 and Windows Server 2025.
- The Azure File Sync agent installation package is specific to the operating system version. If you're upgrading a server to a newer version of Windows Server, you must first uninstall the Azure File Sync agent and restart the server. ***Don't unregister and re-register the server during the OS upgrade, or it will lead to orphaned tiered files on existing server endpoints.*** Then, upgrade the operating system to the new version. After the upgrade is complete, install the Azure File Sync agent that matches the new Windows Server version such as 2016, 2019, 2022, or 2025. After installing the agent on the upgraded server, the Azure portal should reflect the correct server status within 30 minutes.
- The agent requires at least 2 GiB of memory. If the server is running in a virtual machine with dynamic memory enabled, the VM should be configured with a minimum 2048 MiB of memory. See [Recommended system resources](file-sync-planning.md#recommended-system-resources) for more information.
- The agent uses TLS 1.2 or 1.3 (Windows Server 2022 or newer) by default. TLS 1.0 and 1.1 aren't supported.
- Server registration using the [Register-AzStorageSyncServer](/powershell/module/az.storagesync/register-azstoragesyncserver) and ServerRegistration.exe require .NET Framework 4.7.2. or higher.
- The Storage Sync Agent (FileSyncSvc) service doesn't support server endpoints located on a volume that has the system volume information (SVI) directory compressed. If the SVI directory is compressed, the Storage Sync Agent (FileSyncSvc) service will fail to start.
- Agent installation can fail with error **0x80c84111** if required Windows security updates are missing. To prevent this issue, make sure the following updates are installed based on your server version:
  - Windows Server 2016 [Microsoft Update Catalog](https://catalog.update.microsoft.com/Search.aspx?q=cumulative%20windows%20server%202016) (latest cumulative update)
  - Windows Server 2019 [Microsoft Update Catalog](https://catalog.update.microsoft.com/Search.aspx?q=cumulative%20windows%20server%202019) (latest cumulative update)
  - Cumulative updates are released monthly. To deploy the latest update, users can either use Windows Update or manually download it from the [Microsoft Update Catalog](https://catalog.update.microsoft.com). If installing manually, users should review the associated KB article to ensure all prerequisites are met.​ ​​If the Windows Updates aren't installed prior to installing the Azure File Sync agent, the Storage Sync Agent service (FileSyncSvc) will fail to start. To learn more, see the [Azure File Sync troubleshooting documentation](/troubleshoot/azure/azure-storage/files/file-sync/file-sync-troubleshoot). 


### Interoperability
- Antivirus, backup, and other applications that access tiered files can cause undesirable recall unless they respect the offline attribute and skip reading the content of those files. For more information, see [Troubleshoot Azure File Sync](/troubleshoot/azure/azure-storage/file-sync-troubleshoot?toc=/azure/storage/file-sync/toc.json).
- File Server Resource Manager (FSRM) file screens can cause endless sync failures when files are blocked because of the file screen.
- Running sysprep on a server that has the Azure File Sync agent installed isn't supported and can lead to unexpected results. The Azure File Sync agent should be installed after deploying the server image and completing sysprep mini-setup.

### Sync limitations
The following items don't sync, but the rest of the system continues to operate normally:

- Azure File Sync supports all characters that are supported by the [NTFS file system](/windows/win32/fileio/naming-a-file) except invalid surrogate pairs. See [Troubleshooting guide](/troubleshoot/azure/azure-storage/file-sync-troubleshoot-sync-errors?toc=/azure/storage/file-sync/toc.json#handling-unsupported-characters) for more information.
- Paths that are longer than 2,048 characters.
- The system access control list (SACL) portion of a security descriptor that's used for auditing.
- Extended attributes.
- Alternate data streams.
- Reparse points.
- Hard links.
- Compression (if it's set on a server file) isn't preserved when changes sync to that file from other endpoints.
- Any file that's encrypted with EFS (or other user mode encryption) that prevents the service from reading the data.

> [!NOTE]
> Azure File Sync always encrypts data in transit. Data is always encrypted at rest in Azure.

### Server endpoint
- A server endpoint can be created only on an NTFS volume. Azure File Sync doesn't currently support ReFS, FAT, or FAT32.
- Cloud tiering isn't supported on the system volume. To create a server endpoint on the system volume, disable cloud tiering when creating the server endpoint.
- Failover Clustering is supported only with clustered disks, but not with Cluster Shared Volumes (CSVs).
- A server endpoint can't be nested. It can coexist on the same volume in parallel with another endpoint.
- Don't store an OS or application paging file within a server endpoint location.

### Cloud endpoint
- Azure File Sync supports making changes to the Azure file share directly. However, any changes made on the Azure file share first need to be discovered by an Azure File Sync change detection job. A change detection job is initiated for a cloud endpoint once every 24 hours. To immediately sync files that are changed in the Azure file share, use the [Invoke-AzStorageSyncChangeDetection](/powershell/module/az.storagesync/invoke-azstoragesyncchangedetection) PowerShell cmdlet to manually initiate the detection of changes in the Azure file share.
- The storage sync service and/or storage account can be moved to a different resource group, subscription, or Microsoft Entra (formerly Azure AD) tenant. After moving the storage sync service or storage account, you need to give the Microsoft.StorageSync application access to the storage account (see [Ensure Azure File Sync has access to the storage account](/troubleshoot/azure/azure-storage/file-sync-troubleshoot-sync-errors?toc=/azure/storage/file-sync/toc.json#troubleshoot-rbac)).

> [!NOTE]
> When creating the cloud endpoint, the storage sync service and storage account must be in the same Microsoft Entra ID tenant. After you create the cloud endpoint, you can move the storage sync service and storage account to different Microsoft Entra ID tenants.

### Cloud tiering
- If a tiered file is copied to another location by using Robocopy, the resulting file isn't tiered. The offline attribute might be set because Robocopy incorrectly includes that attribute in copy operations.
- When copying files using Robocopy, use the /MIR option to preserve file timestamps. This will ensure older files are tiered sooner than recently accessed files.

## Version 20.0.0.0
The following release notes are for Azure File Sync version 20.0.0.0 (released February 10, 2025). This release contains improvements for the Azure File Sync service and agent. 

### Improvements and issues that are fixed
**General Availability: Managed Identities support for Azure File Sync service and servers**

Azure File Sync now supports system-assigned managed identities for authentication, eliminating the need for [shared keys](../common/storage-account-keys-manage.md) and simplifying security management with Microsoft Entra ID. You can now configure managed identities directly from the Azure portal, making deployments easier and more secure.
 
When managed identities are configured, the system-assigned managed identities will be used for the following scenarios:
- Storage Sync Service authentication to Azure file share
- Registered server authentication to Azure file share
- Registered server authentication to Storage Sync Service

For more information, see: [How to use managed identities with Azure File Sync](file-sync-managed-identities.md).

> [!NOTE]
>The portal experience for general availability will gradually roll out to all regions in the coming weeks.

**Miscellaneous reliability and telemetry improvements for cloud tiering and sync**
### Evaluation Tool
Before deploying Azure File Sync, you should evaluate whether it's compatible with your system using the Azure File Sync evaluation tool. This tool is an Azure PowerShell cmdlet that checks for potential issues with your file system and dataset, such as unsupported OS version. For installation and usage instructions, see [Evaluation Tool](file-sync-planning.md#evaluation-cmdlet) section in the planning guide.

### Agent installation and server configuration
For more information on how to install and configure the Azure File Sync agent with Windows Server, see [Planning for an Azure File Sync deployment](file-sync-planning.md) and [How to deploy Azure File Sync](file-sync-deployment-guide.md).

- The agent installation requires a restart for servers that have an existing Azure File Sync agent installation if the agent version is older than 18.2.0.0.
- The agent installation package must be installed with elevated (admin) permissions.
- The agent isn't supported on Nano Server deployment option.
- The agent is supported only on Windows Server 2016, Windows Server 2019, Windows Server 2022 and Windows Server 2025.
- The Azure File Sync agent installation package is specific to the operating system version. If you're upgrading a server to a newer version of Windows Server, you must first uninstall the Azure File Sync agent and restart the server. ***Don't unregister and re-register the server during the OS upgrade, or it will lead to orphaned tiered files on existing server endpoints.*** Then, upgrade the operating system to the new version. After the upgrade is complete, install the Azure File Sync agent that matches the new Windows Server version such as 2016, 2019, 2022, or 2025. After installing the agent on the upgraded server, the Azure portal should reflect the correct server status within 30 minutes.
- The agent requires at least 2 GiB of memory. If the server is running in a virtual machine with dynamic memory enabled, the VM should be configured with a minimum 2048 MiB of memory. See [Recommended system resources](file-sync-planning.md#recommended-system-resources) for more information.
- The agent uses TLS 1.2 or 1.3 (Windows Server 2022 or newer) by default and TLS 1.0 and 1.1 are not supported.
- Server registration using the [Register-AzStorageSyncServer](/powershell/module/az.storagesync/register-azstoragesyncserver) and ServerRegistration.exe require .NET Framework 4.7.2. or higher
- The Storage Sync Agent (FileSyncSvc) service doesn't support server endpoints located on a volume that has the system volume information (SVI) directory compressed. If the SVI directory is compressed, the Storage Sync Agent (FileSyncSvc) service will fail to start.
- Agent installation can fail with error **0x80c84111** if required Windows security updates are missing. To prevent this issue, make sure the following updates are installed based on your server version:
  - Windows Server 2016 [Microsoft Update Catalog](https://catalog.update.microsoft.com/Search.aspx?q=cumulative%20windows%20server%202016) (latest cumulative update)
  - Windows Server 2019 [Microsoft Update Catalog](https://catalog.update.microsoft.com/Search.aspx?q=cumulative%20windows%20server%202019) (latest cumulative update)
  - Cumulative updates are released monthly. To deploy the latest update, users can either use Windows Update or manually download it from the [Microsoft Update Catalog](https://catalog.update.microsoft.com). If installing manually, users should review the associated KB article to ensure all prerequisites are met.​ ​​If the Windows Updates aren't installed prior to installing the Azure File Sync agent, the Storage Sync Agent service (FileSyncSvc) will fail to start. To learn more, see the [Azure File Sync troubleshooting documentation](/troubleshoot/azure/azure-storage/files/file-sync/file-sync-troubleshoot). 


### Interoperability
- Antivirus, backup, and other applications that access tiered files can cause undesirable recall unless they respect the offline attribute and skip reading the content of those files. For more information, see [Troubleshoot Azure File Sync](/troubleshoot/azure/azure-storage/file-sync-troubleshoot?toc=/azure/storage/file-sync/toc.json).
- File Server Resource Manager (FSRM) file screens can cause endless sync failures when files are blocked because of the file screen.
- Running sysprep on a server that has the Azure File Sync agent installed isn't supported and can lead to unexpected results. The Azure File Sync agent should be installed after deploying the server image and completing sysprep mini-setup.

### Sync limitations
The following items don't sync, but the rest of the system continues to operate normally:

- Azure File Sync supports all characters that are supported by the [NTFS file system](/windows/win32/fileio/naming-a-file) except invalid surrogate pairs. See [Troubleshooting guide](/troubleshoot/azure/azure-storage/file-sync-troubleshoot-sync-errors?toc=/azure/storage/file-sync/toc.json#handling-unsupported-characters) for more information.
- Paths that are longer than 2,048 characters.
- The system access control list (SACL) portion of a security descriptor that's used for auditing.
- Extended attributes.
- Alternate data streams.
- Reparse points.
- Hard links.
- Compression (if it's set on a server file) isn't preserved when changes sync to that file from other endpoints.
- Any file that's encrypted with EFS (or other user mode encryption) that prevents the service from reading the data.

> [!NOTE]
> Azure File Sync always encrypts data in transit. Data is always encrypted at rest in Azure.

### Server endpoint
- A server endpoint can be created only on an NTFS volume. ReFS, FAT, FAT32, and other file systems aren't currently supported by Azure File Sync.
- Cloud tiering isn't supported on the system volume. To create a server endpoint on the system volume, disable cloud tiering when creating the server endpoint.
- Failover Clustering is supported only with clustered disks, but not with Cluster Shared Volumes (CSVs).
- A server endpoint can't be nested. It can coexist on the same volume in parallel with another endpoint.
- Don't store an OS or application paging file within a server endpoint location.

### Cloud endpoint
- Azure File Sync supports making changes to the Azure file share directly. However, any changes made on the Azure file share first need to be discovered by an Azure File Sync change detection job. A change detection job is initiated for a cloud endpoint once every 24 hours. To immediately sync files that are changed in the Azure file share, use the [Invoke-AzStorageSyncChangeDetection](/powershell/module/az.storagesync/invoke-azstoragesyncchangedetection) PowerShell cmdlet to manually initiate the detection of changes in the Azure file share.
- The storage sync service and/or storage account can be moved to a different resource group, subscription, or Microsoft Entra (formerly Azure AD) tenant. After moving the storage sync service or storage account, you need to give the Microsoft.StorageSync application access to the storage account (see [Ensure Azure File Sync has access to the storage account](/troubleshoot/azure/azure-storage/file-sync-troubleshoot-sync-errors?toc=/azure/storage/file-sync/toc.json#troubleshoot-rbac)).

> [!NOTE]
> When creating the cloud endpoint, the storage sync service and storage account must be in the same Microsoft Entra ID tenant. After you create the cloud endpoint, you can move the storage sync service and storage account to different Microsoft Entra ID tenants.

### Cloud tiering
- If a tiered file is copied to another location by using Robocopy, the resulting file isn't tiered. The offline attribute might be set because Robocopy incorrectly includes that attribute in copy operations.
- When copying files using Robocopy, use the /MIR option to preserve file timestamps. This will ensure older files are tiered sooner than recently accessed files.

## Version 19.1.0.0
The following release notes are for Azure File Sync version 19.1.0.0 (released September 3, 2024). This release contains improvements for the Azure File Sync service and agent.

### Improvements and issues that are fixed
**Faster server provisioning and improved disaster recovery for Azure File Sync server endpoints.**

We have reduced the time it takes for the new server endpoint to be ready to use. Prior to the v19 release, when a new server endpoint is provisioned, it could take hours and sometime days for the server to be ready to use. With our latest improvements, we've substantially shortened this duration, ensuring a faster setup process.

The improvement applies to the following scenarios, when the server endpoint location is empty (no files or directories):
- Creating the first server endpoint of new sync topology after data is copied to the Azure File Share.
- Adding a new empty server endpoint to an existing sync topology.

This improvement will be gradually enabled in all regions within the next month. Once the improvement is enabled in your region, you will see a Provisioning steps tab in the portal after server endpoint creation which allows you to easily determine when the server endpoint is ready for use. For more information, see [Create an Azure File Sync server endpoint](file-sync-server-endpoint-create.md#provisioning-steps) documentation.

**Preview: Managed Identities support for Azure File Sync service and servers**  
Azure File Sync support for managed identities eliminates the need for shared keys as a method of authentication by utilizing a system-assigned managed identity provided by Microsoft Entra ID.

When you enable this configuration, the system-assigned managed identities will be used for the following scenarios:
- Storage Sync Service authentication to Azure file share
- Registered server authentication to Azure file share
- Registered server authentication to Storage Sync Service

For more information, see: [How to use managed identities with Azure File Sync (preview)](file-sync-managed-identities.md).

**Sync performance improvements**  
Sync performance has significantly improved for file share migrations and when metadata-only is changed (for example, ACL changes). Performance numbers will be posted when they are available.

**Support for Windows Server 2025**  
The Azure File Sync agent is now supported on Windows Server 2025 (build 26100).

**Miscellaneous reliability and telemetry improvements for cloud tiering and sync**

### Evaluation Tool
Before deploying Azure File Sync, you should evaluate whether it's compatible with your system using the Azure File Sync evaluation tool. This tool is an Azure PowerShell cmdlet that checks for potential issues with your file system and dataset, such as unsupported OS version. For installation and usage instructions, see [Evaluation Tool](file-sync-planning.md#evaluation-cmdlet) section in the planning guide.

### Agent installation and server configuration
For more information on how to install and configure the Azure File Sync agent with Windows Server, see [Planning for an Azure File Sync deployment](file-sync-planning.md) and [How to deploy Azure File Sync](file-sync-deployment-guide.md).

- The agent installation requires a restart for servers that have an existing Azure File Sync agent installation if the agent version is older than 18.2.0.0.
- The agent installation package must be installed with elevated (admin) permissions.
- The agent isn't supported on Nano Server deployment option.
- The agent is supported only on Windows Server 2016, Windows Server 2019, Windows Server 2022 and Windows Server 2025.
- The Azure File Sync agent installation package is specific to the operating system version. If you're upgrading a server to a newer version of Windows Server, you must first uninstall the Azure File Sync agent and restart the server. ***Don't unregister and re-register the server during the OS upgrade, or it will lead to orphaned tiered files on existing server endpoints.*** Then, upgrade the operating system to the new version. After the upgrade is complete, install the Azure File Sync agent that matches the new Windows Server version such as 2016, 2019, 2022, or 2025. After installing the agent on the upgraded server, the Azure portal should reflect the correct server status within 30 minutes.
- The agent requires at least 2 GiB of memory. If the server is running in a virtual machine with dynamic memory enabled, the VM should be configured with a minimum 2048 MiB of memory. See [Recommended system resources](file-sync-planning.md#recommended-system-resources) for more information.
- The agent uses TLS 1.2 or 1.3 (Windows Server 2022 or newer) by default and TLS 1.0 and 1.1 are not supported.
- Server registration using the [Register-AzStorageSyncServer](/powershell/module/az.storagesync/register-azstoragesyncserver) and ServerRegistration.exe require .NET Framework 4.7.2. or higher
- The Storage Sync Agent (FileSyncSvc) service doesn't support server endpoints located on a volume that has the system volume information (SVI) directory compressed. If the SVI directory is compressed, the Storage Sync Agent (FileSyncSvc) service will fail to start.
- Agent installation can fail with error **0x80c84111** if required Windows security updates are missing. To prevent this issue, make sure the following updates are installed based on your server version:
  - Windows Server 2016 [Microsoft Update Catalog](https://catalog.update.microsoft.com/Search.aspx?q=cumulative%20windows%20server%202016) (latest cumulative update)
  - Windows Server 2019 [Microsoft Update Catalog](https://catalog.update.microsoft.com/Search.aspx?q=cumulative%20windows%20server%202019) (latest cumulative update)
  - Cumulative updates are released monthly. To deploy the latest update, users can either use Windows Update or manually download it from the [Microsoft Update Catalog](https://catalog.update.microsoft.com). If installing manually, users should review the associated KB article to ensure all prerequisites are met.​ ​​If the Windows Updates aren't installed prior to installing the Azure File Sync agent, the Storage Sync Agent service (FileSyncSvc) will fail to start. To learn more, see the [Azure File Sync troubleshooting documentation](/troubleshoot/azure/azure-storage/files/file-sync/file-sync-troubleshoot). 

### Interoperability

- Antivirus, backup, and other applications that access tiered files can cause undesirable recall unless they respect the offline attribute and skip reading the content of those files. For more information, see [Troubleshoot Azure File Sync](/troubleshoot/azure/azure-storage/file-sync-troubleshoot?toc=/azure/storage/file-sync/toc.json).
- File Server Resource Manager (FSRM) file screens can cause endless sync failures when files are blocked because of the file screen.
- Running sysprep on a server that has the Azure File Sync agent installed isn't supported and can lead to unexpected results. The Azure File Sync agent should be installed after deploying the server image and completing sysprep mini-setup.

### Sync limitations
The following items don't sync, but the rest of the system continues to operate normally:

- Azure File Sync supports all characters that are supported by the [NTFS file system](/windows/win32/fileio/naming-a-file) except invalid surrogate pairs. See [Troubleshooting guide](/troubleshoot/azure/azure-storage/file-sync-troubleshoot-sync-errors?toc=/azure/storage/file-sync/toc.json#handling-unsupported-characters) for more information.
- Paths that are longer than 2,048 characters.
- The system access control list (SACL) portion of a security descriptor that's used for auditing.
- Extended attributes.
- Alternate data streams.
- Reparse points.
- Hard links.
- Compression (if it's set on a server file) isn't preserved when changes sync to that file from other endpoints.
- Any file that's encrypted with EFS (or other user mode encryption) that prevents the service from reading the data.

> [!NOTE]
> Azure File Sync always encrypts data in transit. Data is always encrypted at rest in Azure.

### Server endpoint
- A server endpoint can be created only on an NTFS volume. ReFS, FAT, FAT32, and other file systems aren't currently supported by Azure File Sync.
- Cloud tiering isn't supported on the system volume. To create a server endpoint on the system volume, disable cloud tiering when creating the server endpoint.
- Failover Clustering is supported only with clustered disks, but not with Cluster Shared Volumes (CSVs).
- A server endpoint can't be nested. It can coexist on the same volume in parallel with another endpoint.
- Don't store an OS or application paging file within a server endpoint location.

### Cloud endpoint
- Azure File Sync supports making changes to the Azure file share directly. However, any changes made on the Azure file share first need to be discovered by an Azure File Sync change detection job. A change detection job is initiated for a cloud endpoint once every 24 hours. To immediately sync files that are changed in the Azure file share, use the [Invoke-AzStorageSyncChangeDetection](/powershell/module/az.storagesync/invoke-azstoragesyncchangedetection) PowerShell cmdlet to manually initiate the detection of changes in the Azure file share.
- The storage sync service and/or storage account can be moved to a different resource group, subscription, or Microsoft Entra (formerly Azure AD) tenant. After moving the storage sync service or storage account, you need to give the Microsoft.StorageSync application access to the storage account (see [Ensure Azure File Sync has access to the storage account](/troubleshoot/azure/azure-storage/file-sync-troubleshoot-sync-errors?toc=/azure/storage/file-sync/toc.json#troubleshoot-rbac)).

> [!NOTE]
> When creating the cloud endpoint, the storage sync service and storage account must be in the same Microsoft Entra ID tenant. After you create the cloud endpoint, you can move the storage sync service and storage account to different Microsoft Entra ID tenants.

### Cloud tiering
- If a tiered file is copied to another location by using Robocopy, the resulting file isn't tiered. The offline attribute might be set because Robocopy incorrectly includes that attribute in copy operations.
- When copying files using Robocopy, use the /MIR option to preserve file timestamps. This will ensure older files are tiered sooner than recently accessed files.

## Version 18.2.0.0
The following release notes are for Azure File Sync version 18.2.0.0 (released July 9, 2024). This release contains improvements for the Azure File Sync agent. These notes are in addition to the release notes listed for version 18.0.0.0 and 18.1.0.0.

### Improvements and issues that are fixed
- Rollup update for Azure File Sync agent [v18](#version-18000) and [v18.1](#version-18100-security-update) releases.
- This release also includes sync reliability improvements.

## Version 18.1.0.0 (Security Update)
The following release notes are for Azure File Sync version 18.1.0.0 (released June 11, 2024). This release contains a security update for servers that have v18 agent version installed. These notes are in addition to the release notes listed for version 18.0.0.0.

### Improvements and issues that are fixed
Fixes an issue that might allow unauthorized users to delete files in locations they don’t have access. This is a security-only update. For more information about this vulnerability, see [CVE-2024-35253](https://msrc.microsoft.com/update-guide/en-US/advisory/CVE-2024-35253).

## Version 18.0.0.0
The following release notes are for Azure File Sync version 18.0.0.0 (released May 8, 2024). This release contains improvements for the Azure File Sync service and agent.

### Improvements and issues that are fixed
**Faster server provisioning and improved disaster recovery for Azure File Sync server endpoints**
We're reducing the time it takes for the new server endpoint to be ready to use. When a new server endpoint is provisioned, it could take hours and sometime days for the server to be ready to use. With our latest improvements, we've substantially shortened this duration for a more efficient setup process.

The improvement applies to the following scenarios, when the server endpoint location is empty (no files or directories):
- Creating the first server endpoint of new sync topology after data is copied to the Azure File Share.
- Adding a new empty server endpoint to an existing sync topology.

How to get started: Sign up for the public preview [here](https://forms.office.com/r/gCLr1PDZKL).

**Sync performance improvements**  
Sync upload performance has improved, and performance numbers will be posted when they are available. This improvement will mainly benefit file share migrations (initial upload) and high churn events on the server in which a large number of files need to be uploaded, for example ACL changes.

**Miscellaneous reliability and telemetry improvements for cloud tiering and sync**

### Evaluation Tool
Before deploying Azure File Sync, you should evaluate whether it's compatible with your system using the Azure File Sync evaluation tool. This tool is an Azure PowerShell cmdlet that checks for potential issues with your file system and dataset, such as unsupported OS version. For installation and usage instructions, see [Evaluation Tool](file-sync-planning.md#evaluation-cmdlet) section in the planning guide.

### Agent installation and server configuration
For more information on how to install and configure the Azure File Sync agent with Windows Server, see [Planning for an Azure File Sync deployment](file-sync-planning.md) and [How to deploy Azure File Sync](file-sync-deployment-guide.md).

- The agent installation package must be installed with elevated (admin) permissions.
- The agent isn't supported on Nano Server deployment option.
- The agent is supported only on Windows Server 2019, Windows Server 2016 and Windows Server 2022.
- The Azure File Sync agent installation package is specific to the operating system version. If you're upgrading a server to a newer version of Windows Server, you must first uninstall the Azure File Sync agent and restart the server. ***Don't unregister and re-register the server during the OS upgrade, or it will lead to orphaned tiered files on existing server endpoints.*** Then, upgrade the operating system to the new version. After the upgrade is complete, install the Azure File Sync agent that matches the new Windows Server version such as 2016, 2019, 2022, or 2025. After installing the agent on the upgraded server, the Azure portal should reflect the correct server status within 30 minutes.
- The agent requires at least 2 GiB of memory. If the server is running in a virtual machine with dynamic memory enabled, the VM should be configured with a minimum 2048 MiB of memory. See [Recommended system resources](file-sync-planning.md#recommended-system-resources) for more information.
- The Storage Sync Agent (FileSyncSvc) service doesn't support server endpoints located on a volume that has the system volume information (SVI) directory compressed. This configuration will lead to unexpected results.
- All supported Azure File Sync agent versions use TLS 1.2 by default and TLS 1.0 and 1.1 are not supported. Starting with v18 agent version TLS 1.3 will be supported for Windows Server 2022.
- Agent installation can fail with error **0x80c84111** if required Windows security updates are missing. To prevent this issue, make sure the following updates are installed based on your server version:
  - Windows Server 2016 [Microsoft Update Catalog](https://catalog.update.microsoft.com/Search.aspx?q=cumulative%20windows%20server%202016) (latest cumulative update)
  - Windows Server 2019 [Microsoft Update Catalog](https://catalog.update.microsoft.com/Search.aspx?q=cumulative%20windows%20server%202019) (latest cumulative update)
  - Cumulative updates are released monthly. To deploy the latest update, users can either use Windows Update or manually download it from the [Microsoft Update Catalog](https://catalog.update.microsoft.com). If installing manually, users should review the associated KB article to ensure all prerequisites are met.​ ​​If the Windows Updates aren't installed prior to installing the Azure File Sync agent, the Storage Sync Agent service (FileSyncSvc) will fail to start. To learn more, see the [Azure File Sync troubleshooting documentation](/troubleshoot/azure/azure-storage/files/file-sync/file-sync-troubleshoot). 


### Interoperability
- Antivirus, backup, and other applications that access tiered files can cause undesirable recall unless they respect the offline attribute and skip reading the content of those files. For more information, see [Troubleshoot Azure File Sync](/troubleshoot/azure/azure-storage/file-sync-troubleshoot?toc=/azure/storage/file-sync/toc.json).
- File Server Resource Manager (FSRM) file screens can cause endless sync failures when files are blocked because of the file screen.
- Running sysprep on a server that has the Azure File Sync agent installed isn't supported and can lead to unexpected results. The Azure File Sync agent should be installed after deploying the server image and completing sysprep mini-setup.

### Sync limitations
The following items don't sync, but the rest of the system continues to operate normally:

- Azure File Sync v17 agent and later supports all characters that are supported by the [NTFS file system](/windows/win32/fileio/naming-a-file) except invalid surrogate pairs. See [Troubleshooting guide](/troubleshoot/azure/azure-storage/file-sync-troubleshoot-sync-errors?toc=/azure/storage/file-sync/toc.json#handling-unsupported-characters) for more information.
- Paths that are longer than 2,048 characters.
- The system access control list (SACL) portion of a security descriptor that's used for auditing.
- Extended attributes.
- Alternate data streams.
- Reparse points.
- Hard links.
- Compression (if it's set on a server file) isn't preserved when changes sync to that file from other endpoints.
- Any file that's encrypted with EFS (or other user mode encryption) that prevents the service from reading the data.

> [!NOTE]
> Azure File Sync always encrypts data in transit. Data is always encrypted at rest in Azure.

### Server endpoint
- A server endpoint can be created only on an NTFS volume. ReFS, FAT, FAT32, and other file systems aren't currently supported by Azure File Sync.
- Cloud tiering isn't supported on the system volume. To create a server endpoint on the system volume, disable cloud tiering when creating the server endpoint.
- Failover Clustering is supported only with clustered disks, but not with Cluster Shared Volumes (CSVs).
- A server endpoint can't be nested. It can coexist on the same volume in parallel with another endpoint.
- Don't store an OS or application paging file within a server endpoint location.

### Cloud endpoint
- Azure File Sync supports making changes to the Azure file share directly. However, any changes made on the Azure file share first need to be discovered by an Azure File Sync change detection job. A change detection job is initiated for a cloud endpoint once every 24 hours. To immediately sync files that are changed in the Azure file share, use the [Invoke-AzStorageSyncChangeDetection](/powershell/module/az.storagesync/invoke-azstoragesyncchangedetection) PowerShell cmdlet to manually initiate the detection of changes in the Azure file share.
- The storage sync service and/or storage account can be moved to a different resource group, subscription, or Microsoft Entra (formerly Azure AD) tenant. After moving the storage sync service or storage account, you need to give the Microsoft.StorageSync application access to the storage account (see [Ensure Azure File Sync has access to the storage account](/troubleshoot/azure/azure-storage/file-sync-troubleshoot-sync-errors?toc=/azure/storage/file-sync/toc.json#troubleshoot-rbac)).

> [!NOTE]
> When creating the cloud endpoint, the storage sync service and storage account must be in the same Microsoft Entra tenant. After you create the cloud endpoint, you can move the storage sync service and storage account to different Microsoft Entra tenants.

### Cloud tiering
- If a tiered file is copied to another location by using Robocopy, the resulting file isn't tiered. The offline attribute might be set because Robocopy incorrectly includes that attribute in copy operations.
- When copying files using Robocopy, use the /MIR option to preserve file timestamps. This will ensure older files are tiered sooner than recently accessed files.

