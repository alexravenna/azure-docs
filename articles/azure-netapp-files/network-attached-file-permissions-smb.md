---
title: Understand SMB file permissions in Azure NetApp Files
description: Learn about SMB file permissions options in Azure NetApp Files.
services: azure-netapp-files
author: b-ahibbard
ms.service: azure-netapp-files
ms.topic: concept-article
ms.date: 01/13/2025
ms.author: anfdocs
# Customer intent: "As a cloud administrator, I want to understand SMB file permissions using NTFS ACLs in Azure NetApp Files, so that I can effectively manage access controls and ownership for my organization's files and folders."
---

# Understand SMB file permissions in Azure NetApp Files 

SMB volumes in Azure NetApp Files can leverage NTFS security styles to make use of NTFS access control lists (ACLs) for access controls. 

NTFS ACLs provide granular permissions and ownership for files and folders by way of access control entries (ACEs). Directory permissions can also be set to enable or disable inheritance of permissions.

:::image type="content" source="./media/network-attached-file-permissions-smb/access-control-entry-diagram.png" alt-text="Diagram of access control entries." lightbox="./media/network-attached-file-permissions-smb/access-control-entry-diagram.png":::

For a complete overview of NTFS-style ACLs, see [Microsoft Access Control overview](/windows/security/identity-protection/access-control/access-control).

## Next steps 

* [Create an SMB volume](azure-netapp-files-create-volumes-smb.md)
