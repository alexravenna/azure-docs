---
title: 'Static routes in Azure Virtual WAN'
titleSuffix: Azure Virtual WAN
description: Learn about the types of static routes in Azure Virtual WAN, their use cases, best practices, and limitations.
author: wtnlee
ms.service: azure-virtual-wan
ms.topic: concept-article
ms.date: 04/27/2026
ms.author: wellee
ms.custom:
---

# Static routes in  Virtual WAN

In Virtual WAN, you can use static routes to direct traffic to a specific next-hop. In Virtual WAN, static routes are used for two main routing use cases: routing traffic through an Azure Firewall deployed in the Virtual WAN hub, or routing traffic to a designated IP address (often a load balancer in front of a Network Virtual Appliance) deployed in a spoke virtual network that's connected to the Virtual WAN hub.

The following document describe the different types of static routes in Virtual WAN, common use cases, and the main best practices and limitations to consider during network design and implementation.

## Routing use cases

### Routing traffic to Azure Firewall

>[!NOTE]
> There are two disjoint ways to direct traffic to Azure Firewall, static routes and [routing intent](how-to-configure-routing-policies.md). This section only covers the static routing approach.

#### Configuration

You can configure Virtual WAN to route traffic to Azure Firewall in secure hub using static routes. This configuration involves adding two separate configurations to your Virtual WAN deployment:

* Add  **static routes** to Virtual WAN hub route tables, with next hop specified as your Azure Firewall resource identifier.
* Configure the **associated** and **propagated route tables** of your Virtual WAN connections.  

To configure static routes and associated/propagated route tables in secure hub scenarios, utilize the following best practices:

* Minimize the number of custom route tables (in addition to **defaultRouteTable** and **noneRouteTable**). Custom route tables should be used for more customized routing scenarios such as different routing patterns for Virtual Networks.
* Use aggregate ranges instead of specific ranges in static routes when possible. This minimizes the number of static routes configured.
* All branches must associate to the **defaultRouteTable** and propagate to the **same set** of route table and route table labels.
* Propagating a connection's routes to a route table implies that all connections associated to that route table can access the propagated routes directly. Ensure your routing configuration is consistent and reuslts in routing symmetry. For example, if branches propagate to a Virtual Network's route table, ensure that the same Virtual Networks propagate to the defaultRouteTable. The same holds if a connection **doesn't** propagate to another connection's route table.
* Similarly, not propgating a connection to a route table typically implies that connections associated to that same route table won't be able to access that connection. A static route is required.

#### Common use cases

One common use case for static routes in Virtual WAN is to send same-hub private traffic through an Azure Firewall deployed in the virtual hub. In this design, the firewall acts as the next hop for traffic that would otherwise flow directly through the hub router.

This pattern is used to deliver Azure Firewall inspection for the following high-level use cases:

* [Traffic between on-premises branches and virtual networks connected to the same virtual hub](static-routes-firewall-basic.md).
* [Traffic between virtual networks that are connected to the same virtual hub](static-routes-firewall-basic.md).
* [Traffic between the virtual hub's locally connected on-premises branches and Virtual Networks and the internet](static-routes-firewall-basic.md).

Additional more complex that can use use cases include:

* [Traffic between certain Virtual Networks should bypass inspection (routed via Virtual Hub router)](firewall-custom-bypass.md#virtual-network-to-virtual-network-selective-inspection).
* [Traffic between certain Virtual Networks and on-premises should bypass inspection](firewall-custom-bypass.md#on-premises-to-virtual-network-selective-inspection).
* [Traffic local to Virtual WAN hub is inspected via Azure Firewall, while inter-hub traffic bypasses inspection](static-routes-firewall-basic.md#local-hub-inspection-with-inter-hub-routed-directly).

Other common use cases that require alternate approaches or are not supported: 

| Use Case| Alternative Approach |
|--|--|
| Route traffic to a NVA deployed inside of the Virtual WAN hub| Use [routing intent and policies](how-to-routing-policies.md).|
| Inspect inter-hub traffic | Use [routing intent and policies](how-to-routing-policies.md).|
| Inspect branch-to-branch traffic (ExpressRoute, Site-to-site VPN and Point-to-site VPN)|Use [routing intent and policies](how-to-routing-policies.md).|
| Virtual Network isolation with secure hubs.| Utilize **Azure Firewall network rules** to block traffic between Virtual Networks that shouldn't be able to communicate. Virtual WAN routing (even with propagations and associations) properly configured, can't guarantee two Virtual Networks are isolated from a routing perspective. For example, two Virtual Networks that don't propagate to each other can still communicate if an aggregate route (such as a 10.0.0.0/8 or 0.0.0.0/0) is configured as a static route on the Virtual Network route table. |

### Routing traffic to a NVA in a spoke Virtual Network

#### Configuration options

You can configure routing to an IP address in a spoke virtual network in two ways:

* **Option 1: Specify the static route on the virtual network connection**. Set **Propagate static route** to **True**. In this model, the static route on the virtual network connection is propagated into the hub so the route can be used without adding a separate static route entry in the Virtual WAN route table. This method of configuration is more scalable, as Virtual WAN will automatically propagate the static routes according to the Virtual Network's propagated route tables and labels.
* **Option 2:** Specify a static route in a Virtual WAN route table, with the next hop set to the **Hub virtual network connection**. In this model, there must also be a corresponding static route on the  virtual network connection that specifies the next hop IP address for the traffic. Additionally, you must add a static route for every remote Virtual WAN hub needs to use a NVA deployed in a local spoke Virtual network.

The two configuration options support different routing patterns and have different available use cases:

| Option | Overview| Supported Use Cases|Example Architectures | Unsupported Use Cases|
|--|--|--|--|--|
| Option 1 | Static route on the virtual network connection with **Propagate static route** set to **True** |Utilize spoke NVA as a source of routes for indirect spokes, VPN tunnels terminated on the NVA device or as an internet edge. Compatible with routing intent hubs. | [Indirect spoke architectures](indirect-spoke-architecture.md) and [route internet-bound traffic to spoke NVA for egress](indirect-spoke-architecture.md), [Hybrid scenarios](hybrid-firewall-spoke-static.md)|  This configuration option **can't be used** for inspection scenarios between two Virtual WAN connections.|
| Option 2 |Static route in a Virtual WAN route table with next hop set to the hub virtual network connection, plus a matching next hop IP on the virtual network connection | Utilize spoke NVA as a source of routes for indirect spokes, VPN tunnels terminated on the NVA device or as an internet edge. Utilize for inspection scenarios between two Virtual WAN connections (on-premises to Virtual Network).  | [Indirect spoke architectures](indirect-spoke-architecture.md), [routing internet-bound traffic to spoke NVA for egress](indirect-spoke-architecture.md), [Hybrid scenarios](hybrid-firewall-spoke-static.md), **[On-premises to Virtual Network inspection](spoke-inspection-north-south.md)**. | Not compatible with hubs using routing intent.|


When using static routes pointing to a Virtual Network connection, note the following best practices and considerations:

* The [bypass next-hop IP address](howto-connect-vnet-hub.md#bypassexplained) setting determines how traffic destined for IP addresses in the same Virtual Network as the NVA are routed. Align this setting with your intended network pattern. Often times, setting this setting to **true** is critical to correctly  NVA management traffic to the expected NVA interface or instance.
* If there are multiple static routes configured where the destination CIDRs  are **not** in IANA RFC1918, all static routes with non-RFC1918 destinations must use the **same next hop IP address**.
* For scenarios where the NVA is used to inspect traffic between on-premises and other Virtual Networks, the NVA's Virtual Network will typically be associated to a **custom** route table diffrent than the branches or other Virtual Networks, while all the other connections will propagate to the NVA Virtual Network's **custom route table**. For an example, see the common use cases section below. 

#### Common use cases

* [Routing traffic to indirect spokes. Indirect spokes are virtual networks that are peered to Virtual WAN spokes, but not directly connected to the Virtual WAN hub](indirect-spoke-architecture.md).
* [Route on-premises traffic destined to a spoke Virtual Network to a NVA deployed in a different spoke VNET for inspection](spoke-inspection-north-south.md).
* [Route internet-bound traffic to a spoke NVA for inspection and egress. Commonly used in scenarios where you don't want to use a Firewall solution directly deployed in the Virtual WAN hub](indirect-spoke-architecture.md).  

Some common use cases that require alternative approaches or are not supported in Virtual WAN:

| Use Case| Alternative Approaches|
|--|--|
| Utilize a NVA to inspect Virtual Network to Virtual Network traffic via a NVA deployed in a third Virtual WAN spoke.| Not supported. Utilize an indirect spoke architecture where spoke VNETs are peered to the NVA spoke and **not** the Virtual WAN hub. Alternatively, deploy a NVA in the [Virtual WAN hub](about-nva-hub.md), connect all spoke Virtual Networks to the Virtual WAN hub and utilize routing intent.| 
|Utilize a NVA to inspect Branch-to-branch traffic via a NVA deployed in a Virtual WAN spoke. | Not supported. Deploy a NVA in the [Virtual WAN hub](about-nva-hub.md) and utilize routing intent.|

### Combining two types of static routes

You can also combine static routes that point to Azure Firewall in the virtual hub with static routes that point to a virtual network connection. This design is useful when you want different next hops for different traffic classes within the same Virtual WAN deployment.

Common use cases include:

* [Use Azure Firewall in the virtual hub to inspect internal traffic between branches and virtual networks, or between virtual networks connected to the same hub, while using an NVA in a spoke virtual network for internet-bound traffic](hybrid-firewall-spoke-static.md).

Other common use cases that typically require an alternative approach are shown in the following table.

| Use case | Alternative approach |
|--|--|
|Double-inspection scenarios: inspect traffic destined for indirect spoke or Internet in secure hub and then NVA in spoke| Use [routing intent and policies](how-to-routing-policies.md) and static routes on Virtual Networks connection with **Propagate static routes** set to **True**. |