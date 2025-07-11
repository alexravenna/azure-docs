---
ms.date: 05/18/2025
ms.topic: include
ms.custom:
  - ignite-2024
  - build-2025
---

### Use Microsoft Entra ID authentication on your cache

Azure Managed Redis caches have Microsoft Entra Authentication enabled by default.

1. In the Azure portal, select the cache where you'd like to use Microsoft Entra token-based authentication.

1. Select **Authentication** from the Resource menu.

1. Select **Select member** and enter the name of a valid user. The user you enter is automatically assigned _Data Owner Access Policy_ by default when you select **Save**. You can also enter a managed identity or service principal to connect to your cache instance.

     :::image type="content" source="media/cache-entra-access/cache-enable-microsoft-entra.png" alt-text="Screenshot showing authentication selected in the resource menu and the enable Microsoft Entra authentication checked.":::

For information on using Microsoft Entra ID with Azure CLI, see the [reference pages for identity](/cli/azure/redis/identity).
