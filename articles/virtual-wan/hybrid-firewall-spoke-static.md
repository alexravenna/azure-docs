---
title: 'Hybrid static routing with Azure Firewall and spoke NVAs'
titleSuffix: Azure Virtual WAN
description: Learn about Azure Virtual WAN routing scenarios that combine Azure Firewall and spoke NVAs with static routing.
author: wtnlee
ms.service: azure-virtual-wan
ms.topic: concept-article
ms.date: 04/28/2026
ms.author: wellee
ms.custom:
---

# Advanced: Combine static routing to Azure Firewall and spoke NVAs

## Scenario overview

> [!NOTE]
> This architecture doesn't support double-inspection scenarios. Double-inspection scenarios are use cases where packets are routed to and inspected by Azure Firewall in the Virtual WAN hub and then forwarded to an NVA in a spoke virtual network. This architecture is supported only with Virtual WAN hubs that use routing intent together with static routes on virtual network connections and **Propagate static route** set to **true**.

This design pattern uses a combination of static routes to send private traffic (traffic between on-premises and virtual networks), to Azure Firewall for inspection. At the same time, internet-bound traffic is sent to a network virtual appliance (NVA) deployed in a spoke virtual network for inspection and breakout.

This design pattern is most common in network topologies where Azure Firewall is used for traffic inspection between Azure and on-premises workloads, while Internet access is controlled by a third-party network virtual appliance (NVA) or  secure web gateway (SWG) service. 

## Network diagram

:::image type="content" source="./media/route-scenarios/hybrid-firewall-spoke.png" alt-text="Diagram that shows a hybrid static routing design using Azure Firewall in the Virtual WAN hub for private traffic inspection and a spoke NVA for internet egress." lightbox="./media/route-scenarios/hybrid-firewall-spoke.png":::

In the above diagram, there are two types of Virtual Networks:

* DMZ Virtual Network: Hosts the NVA or SWG service used for Internet egress.
* Workload Virtual Networks: All other Virtual Networks connected to the Virtual WAN hub.

## Traffic flows

The following connectivity matrix summarizes the traffic flows in this scenario.

| Source | Workload Virtual Network | DMZ Virtual Network | On-premises | Internet |
|--|--|--|--|--|
| Workload Virtual Network |via Azure Firewall | via Azure Firewall | Via Azure Firewall | Via DMZ NVA |
| DMZ Virtual Network | Direct | Direct | Via Azure Firewall | Direct |
| On-premises | Via Azure Firewall | Via Azure Firewall | Direct | Via DMZ NVA |


## Configuration

### Virtual WAN route tables

| Route table name | Associated connections | Reasoning |
|--|--|--|
| defaultRouteTable | branches | Used for branch connectivity and to ensure on-premises traffic is steered to Azure Firewall for private traffic inspection and to the DMZ NVA for internet egress. |
| workloadRouteTable | Workload virtual networks | Used by workload virtual networks to route private traffic to Azure Firewall and internet-bound traffic to the DMZ virtual network. |
| dmzRouteTable | DMZ virtual network | Used by the DMZ virtual network to learn branch and workload prefixes so the NVA can return traffic correctly after inspection and breakout. |

### Virtual WAN routing configuration

| Connection | Associated route table | Propagated route table | Reasoning |
|--|--|--|--|
| On-premises connections | defaultRouteTable | defaultRouteTable, dmzRouteTable | On-premises prefixes must be sent to the DMZ route table so so DMZ can route  traffic directly back to on-premises. |
| Workload virtual networks | workloadRouteTable | workloadRouteTable, dmzRouteTable | Workload prefixes must be visible to both other workloads and the dmzrouteTable. Propagating to DMZ ensures ensures DMZ NVA is able to route traffic directly back to workload Virtual Networks. |
| DMZ virtual network | dmzRouteTable | dmzRouteTable, defaultRouteTable, workloadRouteTable | The DMZ virtual network propagates to all route tables. This ensures all other connections can access the DMZ virtual network directly for egress. |

### Static routes

The following static routes are configured by adding the static routes on the NVA virtual network connection directly, with **Propagate static route** set to **true**. Static routes are automatically injected into the appropraite route tables (defaultRouteTAble and workloadRouteTable)

| Virtual network connection | Address prefix | Next hop IP address | Reasoning |
|--|--|--|--|
| DMZ | 0.0.0.0/0 | 10.5.10.5 | Sends internet-bound traffic to the NVA or SWG deployed in the DMZ virtual network for inspection and breakout. |

The following static routes are needed to route traffic from on-premises and Virtual Networks to Azure Firewall:

| Route Table | Address prefix | Next hop  | Reasoning |
|--|--|--|--|
| workloadRouteTable | 10.0.0.0/8, 192.168.0.0/16, 172.16.0.0/12 | Azure Firewall| Sends Virutal Network traffic destined for on-premises to Azure Firewall for inspection. |
| defaultRouteTable | 10.0.0.0/8, 192.168.0.0/16, 172.16.0.0/12 | Azure Firewall| Sends on-premises traffic destined for workload Virtual Networks to Azure Firewall for inspection. |

If using the **legacy** static routing model, add additional static routes in the route tables:

| Route Table | Address prefix | Next hop  | Reasoning |
|--|--|--|--|
| workloadRouteTable | 0.0.0.0/0 | DMZ Virtual Network connection| Sends Virtual Network to internet traffic to DMZ NVA. |
| defaultRouteTable | 0.0.0.0/0 | DMZ Virtual Network connection| Sends on-premises to internet traffic to DMZ NVA. |


## Additional considerations

* If static routes overlap with the NVA virtual network address space, make sure the **bypass next hop IP for workloads within this VNet** setting is configured correctly on the virtual network connection. Toggling this setting to "true" is often required for scenarios where direct access to the management interface of the NVA is required.  For more information, see [Bypass next hop IP for workloads within this VNet](howto-connect-vnet-hub.md#bypassexplained).
* To ensure a connection learns the 0.0.0.0/0 route from Virtual WAN, ensure the **Propagate default route** or **Enable Internet Security** setting is set to **On** for the connection.

