---
title: 'Configure P2S VPN clients: certificate authentication: OpenVPN Client - Windows'
titleSuffix: Azure VPN Gateway
description: Learn how to configure VPN clients for P2S configurations that use certificate authentication. This article applies to Windows and the OpenVPN Client.
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 02/21/2024
ms.author: cherylmc
---

# Configure OpenVPN client for P2S certificate authentication connections - Windows

If your point-to-site (P2S) VPN gateway is configured to use OpenVPN and certificate authentication, you can connect to your virtual network using the OpenVPN Client. This article walks you through the steps to configure the **OpenVPN client** and connect to your virtual network.

## Before you begin

This article assumes that you've already performed the following prerequisites:

* You created and configured your VPN gateway for point-to-site certificate authentication and the OpenVPN tunnel type. See [Configure server settings for P2S VPN Gateway connections - certificate authentication](vpn-gateway-howto-point-to-site-resource-manager-portal.md) for steps.
* You generated client certificates and downloaded the VPN client configuration files. See [Point-to-site VPN clients: certificate authentication - Windows ](point-to-site-vpn-client-cert-windows.md)

Before beginning client configuration steps, verify that you're on the correct VPN client configuration article. The following table shows the configuration articles available for VPN Gateway point-to-site VPN clients. Steps differ, depending on the authentication type, tunnel type, and the client OS.

[!INCLUDE [All client articles](../../includes/vpn-gateway-vpn-client-install-articles.md)]

### Connection requirements

To connect to Azure, each connecting client computer requires the following items:

* The Open VPN Client software must be installed and configured on each client computer.
* The client computer must have a client certificate that's installed locally.

## View configuration files

The VPN client profile configuration package contains specific folders. The files within the folders contain the settings needed to configure the VPN client profile on the client computer. The files and the settings they contain are specific to the VPN gateway and the type of authentication and tunnel your VPN gateway is configured to use.

Locate and unzip the VPN client profile configuration package you generated. For Certificate authentication and OpenVPN, you should see an OpenVPN folder. If you don't see the folder, verify the following items:

* Verify that your VPN gateway is configured to use the OpenVPN tunnel type.
* If you're using Microsoft Entra authentication, you might not have an OpenVPN folder. See the [Microsoft Entra ID](openvpn-azure-ad-client.md) configuration article instead.

## Configure the client

[!INCLUDE [Configuration steps](../../includes/vpn-gateway-vwan-config-openvpn-windows.md)]

## Next steps

[Point-to-site configuration steps](vpn-gateway-howto-point-to-site-resource-manager-portal.md)
[Point-to-site VPN clients: certificate authentication - Windows ](point-to-site-vpn-client-cert-windows.md)