---
title: Geo-disaster recovery for Azure Event Hubs
description: Learn about the geo-disaster recovery feature in Azure Event Hubs, which replicates namespace metadata to enable failover during regional outages.
ms.topic: concept-article
ms.date: 05/05/2026
#customer intent: As an IT administrator or architect, I want to understand geo-disaster recovery for Azure Event Hubs so that I can plan for regional outages.
---

# Azure Event Hubs geo-disaster recovery

Geo-disaster recovery is a disaster recovery feature in Azure Event Hubs that continuously replicates your namespace configuration (event hubs, consumer groups, and settings) from a primary namespace to a secondary namespace. This feature enables you to initiate a failover from the primary to the secondary namespace during regional outages.

> [!NOTE]
> This article describes the geo-disaster recovery feature that replicates metadata only. For information about the geo-replication feature, which replicates both data and metadata, see [Geo-replication](./geo-replication.md).

The all-active Azure Event Hubs cluster model with [availability zone support](/azure/reliability/reliability-event-hubs) provides resiliency against hardware and datacenter outages. However, if a disaster occurs where an entire region and all zones are unavailable, you can use geo-disaster recovery to recover your workload and application configuration.

The concepts and workflow described in this article apply to disaster scenarios, not temporary outages. For a detailed discussion of disaster recovery in Microsoft Azure, see [Disaster recovery for Azure applications](/azure/architecture/resiliency/disaster-recovery-azure-applications). With geo-disaster recovery, you can initiate a once-only failover move from the primary to the secondary at any time. The failover move points the chosen alias name for the namespace to the secondary namespace. After the move, the pairing is removed. The failover is nearly instantaneous once initiated.

> [!IMPORTANT]
> - The feature enables instantaneous continuity of operations with the same configuration, but **does not replicate the event data**. Unless the disaster caused the loss of all zones, the event data that is preserved in the primary Event Hub after failover will be recoverable and the historic events can be obtained from there once access is restored. For replicating event data and operating corresponding namespaces in active/active configurations to cope with outages and disasters, don't lean on this geo-disaster recovery feature set, but follow the [replication guidance](event-hubs-federation-overview.md).
> - Microsoft Entra role-based access control (RBAC) assignments to entities in the primary namespace aren't replicated to the secondary namespace. Create role assignments manually in the secondary namespace to secure access to them.

To set up geo-disaster recovery pairing and initiate failover, see [Configure geo-disaster recovery](configure-geo-disaster-recovery.md).

## Basic concepts and terms

The disaster recovery feature implements metadata disaster recovery and relies on primary and secondary disaster recovery namespaces. The geo-disaster recovery feature is available for the [standard, premium, and dedicated tiers](https://azure.microsoft.com/pricing/details/event-hubs/) only. You don't need to make any connection string changes, as the connection is made via an alias.

The following terms are used in this article:

- **Alias**: The name for a disaster recovery configuration that you set up. The alias provides a single stable Fully Qualified Domain Name (FQDN) connection string. Applications use this alias connection string to connect to a namespace.
- **Primary/secondary namespace**: The namespaces that correspond to the alias. The primary namespace is active and it receives messages (can be an existing or a new namespace). The secondary namespace is passive and doesn't receive messages. The metadata between both is in sync, so both can seamlessly accept messages without any application code or connection string changes. To ensure that only the active namespace receives messages, you must use the alias.
- **Metadata**: Entities such as event hubs and consumer groups, and their properties of the service that are associated with the namespace. Only entities and their settings are replicated automatically. Messages and events aren't replicated.
- **Failover**: The process of activating the secondary namespace.

## Supported namespace pairs

The following combinations of primary and secondary namespaces are supported:

| Primary namespace tier | Allowed secondary namespace tier |
| ----------------- | -------------------- |
| Standard | Standard, Dedicated |
| Premium | Premium |
| Dedicated | Dedicated |

> [!IMPORTANT]
> You can't pair namespaces that are in the same dedicated cluster. You can pair namespaces that are in separate clusters.

## Failover considerations

Note the following considerations when planning for failover:

- By design, Event Hubs geo-disaster recovery doesn't replicate data, and therefore you can't reuse the old offset value of your primary event hub on your secondary event hub. We recommend restarting your event receiver with one of the following methods:

   - *EventPosition.FromStart()* - If you wish to read all data on your secondary event hub.
   - *EventPosition.FromEnd()* - If you wish to read all new data from the time of connection to your secondary event hub.
   - *EventPosition.FromEnqueuedTime(dateTime)* - If you wish to read all data received in your secondary event hub starting from a given date and time.

- In your failover planning, you should also consider the time factor. For example, if you lose connectivity for longer than 15 to 20 minutes, you might decide to initiate the failover.

- The fact that no data is replicated means that current active sessions aren't replicated. Additionally, duplicate detection and scheduled messages might not work. New sessions, scheduled messages, and new duplicates will work.

- Failing over a complex distributed infrastructure should be [rehearsed](/azure/architecture/reliability/disaster-recovery#disaster-recovery-plan) at least once.

- Synchronizing entities can take some time, approximately 50-100 entities per minute.

- Some aspects of the management plane for the secondary namespace become read-only while geo-recovery pairing is active.

- The data plane of the secondary namespace is read-only while geo-recovery pairing is active. The data plane of the secondary namespace accepts GET requests to enable validation of client connectivity and access controls.

## Private endpoints

This section provides considerations when using geo-disaster recovery with namespaces that use private endpoints. To learn about using private endpoints with Event Hubs in general, see [Configure private endpoints](private-link-service.md).

### New pairings

If you try to create a pairing between a primary namespace with a private endpoint and a secondary namespace without a private endpoint, the pairing fails. The pairing succeeds only if both primary and secondary namespaces have private endpoints. We recommend that you use the same configurations on the primary and secondary namespaces and on virtual networks in which private endpoints are created.

> [!NOTE]
> When you try to pair the primary namespace with a private endpoint and a secondary namespace, the validation process only checks whether a private endpoint exists on the secondary namespace. It doesn't check whether the endpoint works or will work after failover. It's your responsibility to ensure that the secondary namespace with private endpoint works as expected after failover.
>
> To test that the private endpoint configurations are the same on primary and secondary namespaces, send a read request (for example: [Get Event Hub](/rest/api/eventhub/get-event-hub)) to the secondary namespace from outside the virtual network, and verify that you receive an error message from the service.

### Existing pairings

If pairing between primary and secondary namespace already exists, private endpoint creation on the primary namespace fails. To resolve, create a private endpoint on the secondary namespace first and then create one for the primary namespace.

> [!NOTE]
> While we allow read-only access to the secondary namespace, updates to the private endpoint configurations are permitted.

### Recommended configuration

When creating a disaster recovery configuration for your application and Event Hubs namespaces, you must create private endpoints for both primary and secondary Event Hubs namespaces against virtual networks hosting both primary and secondary instances of your application.

Let's say you have two virtual networks: `VNET-1`, `VNET-2` and these primary and secondary namespaces: `EventHubs-Namespace1-Primary`, `EventHubs-Namespace2-Secondary`. You need to do the following steps:

- On `EventHubs-Namespace1-Primary`, create two private endpoints that use subnets from `VNET-1` and `VNET-2`
- On `EventHubs-Namespace2-Secondary`, create two private endpoints that use the same subnets from `VNET-1` and `VNET-2`

![Private endpoints and virtual networks](./media/event-hubs-geo-dr/private-endpoints-virtual-networks.png)

Advantage of this approach is that failover can happen at the application layer independent of Event Hubs namespace. Consider the following scenarios:

**Application-only failover:** Here, the application won't exist in `VNET-1` but will move to `VNET-2`. As both private endpoints are configured on both `VNET-1` and `VNET-2` for both primary and secondary namespaces, the application will just work.

**Event Hubs namespace-only failover**: Here again, since both private endpoints are configured on both virtual networks for both primary and secondary namespaces, the application will just work.

> [!NOTE]
> For guidance on geo-disaster recovery of a virtual network, see [Virtual Network - Business Continuity](../virtual-network/virtual-network-disaster-recovery-guidance.md).

## Role-based access control

Microsoft Entra role-based access control (RBAC) assignments to entities in the primary namespace aren't replicated to the secondary namespace. Create role assignments manually in the secondary namespace to secure access to them.

## Related content

- [Configure geo-disaster recovery](configure-geo-disaster-recovery.md)
- [Geo-replication](geo-replication.md)
- [.NET GeoDR sample](https://github.com/Azure/azure-event-hubs/tree/master/samples/Management/DotNet/GeoDRClient)
- [Java GeoDR sample](https://github.com/Azure-Samples/eventhub-java-manage-event-hub-geo-disaster-recovery)
