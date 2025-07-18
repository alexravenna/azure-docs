---
title: Delete backups of an Azure NetApp Files volume
description: Describes how to delete individual backups that you no longer need to keep for a volume.
services: azure-netapp-files
author: b-hchen
ms.service: azure-netapp-files
ms.topic: how-to
ms.date: 05/16/2025
ms.author: anfdocs
ms.custom:
  - build-2025
# Customer intent: As a cloud administrator, I want to delete unnecessary backups of an Azure NetApp Files volume, so that I can free up storage space and potentially reduce costs associated with backups.
---
# Delete backups of a volume 

You can delete individual backups that you no longer need to keep for a volume. Deleting backups deletes the associated objects in your Azure Storage account, resulting in space saving.  

By design, Azure NetApp Files prevents you from deleting the latest backup. If the latest backup consists of multiple snapshots taken at the same time (for example, the same daily and weekly schedule configuration), they're all considered as the latest snapshot; deleting the snapshots with the same time is prevented.

Deleting the latest backup is permitted only when either of the following conditions are met:

* There's no backup policy assigned to the volume. 
* The volume has been deleted.
* The latest backup is the only remaining backup for the deleted volume.

If you need to delete backups to free up space, select an older backup from the **Backups** list to delete. The impact of deleting a backup on billing depends on the backup and might marginally reduce costs. Backups are incremental, meaning unchanged data isn't backed up again. This efficient data management results in multiple incremental, small backups. Cost reduction depends on the size of the backup deleted. Deleting larger backups results in greater cost savings.

The first backup contains all data that was on the volume at the time the backup was created. For more information about backups, see [Understand Azure NetApp Files backups](backup-introduction.md) and [Requirements and considerations for Azure NetApp Files backup](backup-requirements-considerations.md).


> [!NOTE]
> Deleting the last backup on a volume removes the reference point for future incremental backups.

>[!IMPORTANT]
>Deleted backups aren't recoverable.

## Steps

>[!IMPORTANT]
>For volumes with existing backups, you can't perform any operations with a backup until you migrate the backups to a backup vault. For more information about this procedure, see [Manage backup vaults](backup-vault-manage.md).

1. Select **Volumes**.
2. Navigate to **Backups**.
3. From the backup list, select the backup to delete. Select the three dots (`…`) to the right of the backup then **Delete** from the Action menu.

    ![Screenshot that shows the Delete menu for backups.](./media/backup-delete/backup-action-menu-delete.png)

## Next steps  

* [Understand Azure NetApp Files backup](backup-introduction.md)
* [Requirements and considerations for Azure NetApp Files backup](backup-requirements-considerations.md)
* [Resource limits for Azure NetApp Files](azure-netapp-files-resource-limits.md)
* [Configure policy-based backups](backup-configure-policy-based.md)
* [Configure manual backups](backup-configure-manual.md)
* [Manage backup policies](backup-manage-policies.md)
* [Search backups](backup-search.md)
* [Restore a backup to a new volume](backup-restore-new-volume.md)
* [Volume backup metrics](azure-netapp-files-metrics.md#volume-backup-metrics)
* [Azure NetApp Files backup FAQs](faq-backup.md)
