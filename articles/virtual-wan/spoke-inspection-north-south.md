---
title: 'Inspect private and internet traffic with spoke NVAs'
titleSuffix: Azure Virtual WAN
description: Learn about Azure Virtual WAN routing scenarios that use spoke NVAs to inspect private and internet-bound traffic.
author: wtnlee
ms.service: azure-virtual-wan
ms.topic: concept-article
ms.date: 04/28/2026
ms.author: wellee
ms.custom:
---

# Advanced: north-south inspection with NVAs

## Scenario overview

This document covers two specific design patterns:

* **North-south branch-to virtual network via Shared services, DMZ for internet egress**: In this design, Virtual WAN routes private traffic that flows between branches and directly connected virtual networks to a spoke NVA that is dedicated to north-south traffic inspection. Internet-bound traffic from spoke Virtual Networks is routed directly to a separate spoke NVA that is dedicated to internet egress.
* **Single spoke NVA for north-south and internet egress**: This pattern is useful when you want to use a single NVAs for centralized inspection of north-south traffic and internet egress and for Virtual WAN to manage all routing. In this design pattern, there are no additional Virtual Network peerings or user-defined routes needed.



## Design Pattern 1: Direct peering between Workloads and DMZ for internet egress

In this option, workload virtual networks are directly peered to the DMZ virtual network for internet egress, and user-defined routes are applied on workload virtual networks to route internet traffic to the DMZ virtual network. Traffic between workload virtual networks and on-premises are inspected by the shared services NVA.

### Network diagram

:::image type="content" source="./media/route-scenarios/spoke-option-direct-peering.png" alt-text="Diagram that shows Option 1 routing with a shared services NVA for private traffic inspection and a DMZ NVA for internet egress." lightbox="./media/route-scenarios/spoke-option-direct-peering.png":::

In the diagram above, there are a few special types of Virtual Networks:
* DMZ Virtual Network: hosts NVA used for internet egress
* Shared Services Virtual Network: hosts NVA used for north-south traffic inspection
* Workload Virtual Network: any other Virtual Networks connected to Virtual WAN hub.


### Traffic flows

The following sections explain how traffic is routed to indirect spokes and the Internet using Virtual WAN static routes.

| Source | Destination | Routing |
|--|--|--|
| Workload Virtual Network | Shared Services Virtual Network | Direct|
| Workload Virtual Network | Workload  Virtual Network | Direct|
| Workload Virtual Network | Branches | Via Shared Services NVA |
| Workload Virtual Network | Internet | Via DMZ NVA (direct peering) |
| Branches | Shared Services Virtual Network | Direct |
| Branches | Workload Virtual Network | Via Shared Services NVA |

### Virtual WAN Route Tables

The architecture needs three different Virtual WAN route tables as there are three connectivity patterns.

| Route Table Name| Associated Connections| Reasoning|
|--|--|--|
|defaultRouteTable| branches| Branches must utilize the defaultRouteTable to route workload Virtual Network traffic to Shared Services NVA.|
|workloadRouteTable| Workload virtual networks| Used by workload virtual networks to route on-premises traffic to Shared Services Virtual NVA.|
|sharedRouteTable | Shared Service and DMZ Virtual Networks|Used by Shared Service and DMZ NVAs to route on-premises and workload virtual network traffic to the appropriate destination. |
 

### Virtual WAN Routing configuration 

| Connection | Associated route table | Propagated route table | Reasoning |
|--|--|--|--|
| Branches | defaultRouteTable | defaultRouteTable, dmzRouteTable | Branches must propagate to the DMZ route table so the DMZ NVA learns branch prefixes and can return north-south and internet-bound traffic correctly. Branches also propagate to the default route table for standard branch reachability. Branches do **not** propagate to Virtual Networks to ensure north-south traffic is inspected by DMZ NVA.|
| Workload Virtual Networks | workloadRouteTable | workloadRouteTable, dmzRouteTable | Workload virtual networks must propagate to the DMZ route table so the DMZ NVA learns workload prefixes and can return both Virtual Network and internet traffic correctly. Workload virtual networks also propagate to the workload route table for workload-to-workload reachability. Workload VNETs do **not** propagate to defaultRotueTable to ensure north-south traffic is inspected by DMZ NVA. |
| DMZ Virtual Network | dmzRouteTable | dmzRouteTable, defaultRouteTable, workloadRouteTable | The DMZ virtual network must propagate to both the branch-facing and workload-facing route tables because the DMZ NVA is the centralized point for north-south traffic inspection and internet egress in this option. |

### Static Routes

The following static routes are configured by adding the static routes on the NVA virtual network connection directly, with **Propagate static route** set to **true**. Static routes are automatically injected into the appropraite route tables (defaultRouteTAble and workloadRouteTable)

|Virtual Network connection| Address Prefix| Next hop IP address| Reasoning|
|--|--|--|--|
|DMZ |10.1.0.0/16 |10.4.0.5 | The shared services Virtual Network needs to attract traffic destined for  workload  prefix ranges. In this simple example, aggregate routes covering workload ranges are used.|
|DMZ |10.2.0.0/16 |10.4.0.5 | The shared services Virtual Network needs to attract traffic destined for branch prefix ranges. In this simple example, aggregate routes covering branch ranges are used.|

Alternatively you can also use the following configuration if utilizing the legacy workflow:

|Route Table| Destination | Next Hop| Next Hop IP (configured on Virtual Network connection)| Reasoning|
|--|--|--|--|--|
| defaultRouteTable|10.1.0.0/16|Shared Services|10.4.0.5| Aggregate range to ensure branches route to workload VNETs via Shared Service NVA.| 
| workloadRouteTable|10.2.0.0/16|Shared Services|10.4.0.5|Aggregate range to ensure workload VNETs route to branches via Shared Service NVA.|

### Additional workload virtual network configurations

Virtual WAN only manages routing on virtual networks that are directly connected to the Virtual WAN hub. Add user-defined routes on the workload virtual networks to internet route traffic to the DMZ Virutal Network.

In the example above, workload Virtual Networks would need the following entries: 

| Prefix | Next Hop IP address | Purpose|
|--|--|--|
|0.0.0.0/0| 10.5.10.5| Route internet traffic for inspection|

The return packets from the Internet is routed directly from the DMZ NVA to the workload Virtual Networks via Virtual Network peering. 


## Design Pattern 2: Utilize Virtual WAN routing to route traffic to DMZ

In this design attern, workload virtual networks are **not** directly peered to the DMZ virtual network for internet egress. Instead, Virtual WAN routing is used to send internet traffic from workload virtual networks and on-premises to the DMZ virtual network NVA for inspection. Traffic between workload virtual networks and on-premises are still inspected by the DMZ NVA.


## Traffic flows

The following sections explain how traffic is routed to indirect spokes and the Internet using Virtual WAN static routes.

| Source | Destination | Routing |
|--|--|--|
| Workload Virtual Network | Shared Services Virtual Network | Direct|
| Workload Virtual Network | Branches | Via Shared Services NVA |
| Workload Virtual Network | Internet | Via DMZ NVA |
| Branches | Shared Services Virtual Network | Direct |
| Branches | Workload Virtual Network | Via Shared Services NVA |
| Branches | Internet | via DMZ NVA  |

### Network diagram

:::image type="content" source="./media/route-scenarios/spoke-option-routing.png" alt-text="Diagram that shows Option 2 routing with Virtual WAN sending both private and internet-bound traffic through a DMZ NVA." lightbox="./media/route-scenarios/spoke-option-routing.png":::

In the diagram above, there are a two special types of Virtual Networks:
* DMZ Virtual Network: hosts NVA used for internet egress and north-south traffic inspection.
* Workload Virtual Network: any other Virtual Networks connected to Virtual WAN hub.

### Traffic flows

The following sections explain how traffic is routed to indirect spokes and the Internet using Virtual WAN static routes.

| Source | Destination | Routing |
|--|--|--|
| Workload Virtual Network | DMZ Virtual Network | Direct|
| Workload Virtual Network | Branches | Via DMZ NVA |
| Workload Virtual Network | Workload Virtual NETwork| Direct |
| Workload Virtual Network | Internet | Via DMZ NVA |
| Branches | Shared Services Virtual Network | Direct |
| Branches | Workload Virtual Network | Via DMZ NVA |
| Branches| Internet| Via DMZ NVA|

### Virtual WAN Route Tables

The architecture needs three different Virtual WAN route tables as there are three connectivity patterns.

| Route Table Name| Associated Connections| Reasoning|
|--|--|--|
|defaultRouteTable| branches| Branches must utilize the defaultRouteTable to route workload Virtual Network and Internet-bound traffic to DMZ NVA.|
|workloadRouteTable| Workload virtual networks| Used by workload virtual networks to route on-premises traffic and Interet-bound traffic to DMZ NVA.|
|dmzRouteTable |  DMZ Virtual Network|Used by DMZ NVAs to route on-premises and workload virtual network traffic to the appropriate destination. |
 

### Virtual WAN Routing configuration 

| Connection | Associated route table | Propagated route table | Reasoning |
|--|--|--|--|
| Branches | defaultRouteTable | defaultRouteTable, dmzRouteTable | Branches must propagate to the DMZ route table so the DMZ NVA learns branch prefixes and can return both private and internet-bound traffic correctly. Branches also propagate to the default route table for standard branch reachability. |
| Workload Virtual Networks | workloadRouteTable | workloadRouteTable, dmzRouteTable | Workload virtual networks must propagate to the DMZ route table so the DMZ NVA learns workload prefixes and can return both private and internet-bound traffic correctly. Workload virtual networks also propagate to the workload route table for workload-to-workload reachability. |
| DMZ Virtual Network | dmzRouteTable | dmzRouteTable, defaultRouteTable, workloadRouteTable | The DMZ virtual network must propagate to both the branch-facing and workload-facing route tables because the DMZ NVA is the centralized point for private traffic inspection and internet egress in this option. |

### Static Routes

The following static routes are configured by adding the static routes on the NVA virtual network connection directly, with **Propagate static route** set to **true**. Static routes are automatically injected into the appropraite route tables (defaultRouteTAble and workloadRouteTable)

|Virtual Network connection| Address Prefix| Next hop IP address| Reasoning|
|--|--|--|--|
|Shared Services|10.1.0.0/16 |10.4.0.5 | The DMZ Virtual Network needs to attract traffic destined for  workload  prefix ranges. In this simple example, aggregate routes covering workload ranges are used.|
|Shared Services|10.2.0.0/16 |10.4.0.5 | The DMZ Virtual Network needs to attract traffic destined for branch prefix ranges. In this simple example, aggregate routes covering branch ranges are used.|
|DMZ |0.0.0.0/0 |10.4.0.5 | The DMZ Virtual Network needs to attract traffic Internet traffic.|


Alternatively you can also use the following configuration if utilizing the legacy workflow:

|Route Table| Destination | Next Hop| Next Hop IP (configured on Virtual Network connection)| Reasoning|
|--|--|--|--|--|
| defaultRouteTable|10.1.0.0/16|DMZ|10.4.0.5| Aggregate range to ensure branches route to workload VNETs via DMZ NVA.| 
| defaultRouteTable|0.0.0.0/0|DMZ|10.4.0.5| Default route to ensure branch to Internet traffic is routed via DMZ NVA.| 
| workloadRouteTable|10.2.0.0/16|DMZ|10.4.0.5|Aggregate range to ensure workload VNETs route to branches via DMZ NVA.|
| workloadRouteTable|0.0.0.0/0|DMZ|10.4.0.5| Default route to ensure branch to workload Virtual Network to Internet traffic is routed via DMZ NVA.|

## Additional considerations

* If static routes overlap with the NVA virtual network address space, make sure the **bypass next hop IP for workloads within this VNet** setting is configured correctly on the virtual network connection. Toggling this setting to "true" is often required for scenarios where direct access to the management interface of the NVA is required.  For more information, see [Bypass next hop IP for workloads within this VNet](howto-connect-vnet-hub.md#bypassexplained).
* To ensure a connection learns the 0.0.0.0/0 route from Virtual WAN, ensure the **Propagate default route** or **Enable Internet Security** setting is set to **On** for the connection.