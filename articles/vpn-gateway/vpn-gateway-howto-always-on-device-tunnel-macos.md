---
title: 'Configure an Always-On VPN user tunnel for macOS'
titleSuffix: Azure VPN Gateway
description: Learn how to configure an Always On VPN user tunnel for your VPN gateway on macOS.
author: flapinski
ms.service: azure-vpn-gateway
ms.topic: how-to
ms.date: 02/02/2026
ms.author: flapinski

# Customer intent: As a network administrator, I want to configure an Always On VPN user tunnel, so that I can maintain persistent, secure connections for remote users without manual intervention.
---
# Configure an Always On VPN user tunnel for macOS

[!INCLUDE [intro](../../includes/vpn-gateway-vwan-always-on-intro.md)]

This article helps you configure an Always On VPN user tunnel for macOS. For information about configuring a device tunnel, see [Configure an Always On VPN device tunnel](vpn-gateway-howto-always-on-device-tunnel.md).

## Configure the gateway

 Use the instructions in the [Configure a Point-to-Site VPN connection](point-to-site-certificate-gateway.md) article to configure the VPN gateway to use IKEv2 and certificate-based authentication.

## Configure a user tunnel

[!INCLUDE [user configuration](../../includes/vpn-gateway-vwan-always-on-device-macos.md)]

## To remove a profile

To remove a profile, use the following steps:
1. Open the Azure VPN Client for Mac.
1. Disconnect the connection, and clear the **Connect automatically** check box (this can also be managed in settings). (Image here)
1. Open the '...' menu next to the profile name, and select **Remove**

## Next steps

To troubleshoot any connection issues that might occur, see [Azure point-to-site connection problems](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md).
