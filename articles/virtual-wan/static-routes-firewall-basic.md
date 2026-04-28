---
title: 'Basic static route scenarios with Azure Firewall in Virtual WAN'
titleSuffix: Azure Virtual WAN
description: Learn about common static route scenarios that use Azure Firewall in an Azure Virtual WAN hub.
author: wtnlee
ms.service: azure-virtual-wan
ms.topic: concept-article
ms.date: 04/27/2026
ms.author: wellee
ms.custom:
---

# Static routes with Azure Firewall in Virtual WAN

## Overview

This document summarizes basic scenarios regarding how to route traffic to route Virtual WAN traffic to Azure Firewall using static routes. The document also contains some notes on how Azure Firewall Manager configures routing when you secure both private and internet traffic with Azure Firewall Manager and set secure interhub to **Off** as well as how Azure Firewall Manager decides connections are **secure** or **not secured**. 

## Private traffic inspection: Branch-to-Virtual Network and Virtual Network-to-Virtual Network via Azure Firewall

> [!NOTE]
> In this configuration, Azure Firewall Manager expects the defaultRouteTable to have a static route named **private_traffic**.

### Traffic patterns

* Private (Virtual Network and on-premises) inspected by Azure Firewall.

### Configuration

Connection routing properties:

| Connection type | Associated route table | Propagated route table(s) | Propagated route labels |
|--|--|--|--|
| Branch connections | defaultRouteTable | - | none |
| Virtual network connections | defaultRouteTable | - | none |

### Virtual WAN route table: defaultRouteTable

| Destination Prefix | Next Hop |
|--|--|
| 10.0.0.0/8, 192.168.0.0/16, 172.16.0.0/12 | Azure Firewall in the local hub |

## Internet traffic inspection by Azure Firewall

> [!NOTE]
> In this configuration, Azure Firewall Manager expects the defaultRouteTable to have a route named **internet_traffic**. Additionally, for a connection to learn the default route (0.0.0.0/0) the Enable Internet Security setting or Propagate default route setting must be set to **true**. Azure Firewall manager using this setting to display whether a connection's Internet traffic is **secured**. 

### Traffic patterns

* Internet traffic is inspected by Azure Firewall.


### Configuration

| Connection type | Associated route table | Propagated route table(s) | Propagated route labels |
|--|--|--|--|
| Branch connections | defaultRouteTable | defaultRouteTable | - |
| Virtual network connections | defaultRouteTable | defaultRouteTable | - |

### Virtual WAN route table: defaultRouteTable

| Destination Prefix | Next Hop |
|--|--|
| 0.0.0.0/0 | Azure Firewall in the local hub |

## Private and Internet traffic inspection

### Traffic patterns

* Private (Virtual Network and on-premises) traffic is inspected by Azure Firewall.
* Internet traffic is inspected by Azure Firewall.

### Configuration

| Connection type | Associated route table | Propagated route table(s) | Propagated route labels |
|--|--|--|--|
| Branch connections | defaultRouteTable | - | none |
| Virtual network connections | defaultRouteTable | - | none |

### Virtual WAN route table: defaultRouteTable

> [!NOTE]
> In this configuration, Azure Firewall Manager expects the defaultRouteTable to have a route named **all_traffic**.

| Destination Prefix | Next Hop |
|--|--|
| 10.0.0.0/8, 192.168.0.0/16, 172.16.0.0/12, 0.0.0.0/0 | Azure Firewall in the local hub |

## Local hub inspection with inter-hub routed directly

Use [routing intent and policies](how-to-routing-policies.md) to inspect inter-hub traffic.  

### Traffic patterns

* Inter-hub traffic bypasses Azure Firewall (routed directly) via Virtual WAN hub.
* Local (same-hub) traffic between Virtual Networks and on-premises inspected by Azure Firewall.
* Internet traffic uses the local Azure Firewall for inspection and breakout.

> [!NOTE]
> Use Virtual WAN route table labels to group hubs across the Virtual WAN to make configurations easier and more scalable. Additionally, this set-up is **not** configurable via Azure Firewall Manager.


### Configuration Hub 1

| Connection type | Associated route table | Propagated route table(s) | Propagated route labels |
|--|--|--|--|
| Branch connections | defaultRouteTable | defaultRouteTable (Hub 2) | - |
| Virtual network connections | defaultRouteTable | defaultRouteTable (Hub 2) | - |

### Virtual WAN route table Hub 1: defaultRouteTable

| Destination Prefix | Next Hop |
|--|--|
|  10.0.0.0/8, 192.168.0.0/16, 172.16.0.0/12, 0.0.0.0/0 | Azure Firewall in Hub 1 |

### Configuration Hub 2

| Connection type | Associated route table | Propagated route table(s) | Propagated route labels |
|--|--|--|--|
| Branch connections | defaultRouteTable | defaultRouteTable (Hub 1) | - |
| Virtual network connections | defaultRouteTable | defaultRouteTable (Hub 1) | - |

### Virtual WAN route table Hub 2: defaultRouteTable

| Destination Prefix | Next Hop |
|--|--|
|  10.0.0.0/8, 192.168.0.0/16, 172.16.0.0/12, 0.0.0.0/0 | Azure Firewall in Hub 2 |