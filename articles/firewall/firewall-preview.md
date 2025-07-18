---
title: Azure Firewall preview features
description: Learn about Azure Firewall preview features that are publicly available now.
services: firewall
author: duongau
ms.service: azure-firewall
ms.topic: concept-article
ms.date: 04/02/2025
ms.author: duau
# Customer intent: "As a network administrator, I want to explore and test Azure Firewall preview features, so that I can enhance our security configurations and monitor health more effectively before they reach general availability."
---

# Azure Firewall preview features

The following Azure Firewall preview features are available publicly for you to deploy and test. Some of the preview features are available on the Azure portal, and some are only visible using a feature flag.

> [!IMPORTANT]
> These features are currently in PREVIEW.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## Feature flags

As new features are released to preview, some of them are behind a feature flag. To enable the functionality in your environment, you must enable the feature flag on your subscription. These features are applied at the subscription level for all firewalls (virtual network firewalls and SecureHub firewalls).  

This article is updated to reflect the features that are currently in preview with instructions to enable them. When the features move to General Availability (GA), they're available to all customers without the need to enable a feature flag. 

## Preview features

The following features are available in preview.

### Explicit proxy (preview)

With the Azure Firewall Explicit proxy set on the outbound path, you can configure a proxy setting on the sending application (such as a web browser) with Azure Firewall configured as the proxy. As a result, traffic from a sending application goes to the firewall's private IP address, and therefore egresses directly from the firewall without using a user defined route (UDR).

For more information, see [Azure Firewall Explicit proxy (preview)](explicit-proxy.md).

### Resource Health (preview)

With the Azure Firewall Resource Health check, you can now diagnose and get support for service problems that affect your Azure Firewall resource. Resource Health allows IT teams to receive proactive notifications on potential health degradations, and recommended mitigation actions per each health event type.  The resource health is also available in a dedicated page in the Azure portal resource page.
Starting in August 2023, this preview is automatically enabled on all firewalls and no action is required to enable this functionality.
For more information, see [Resource Health overview](/azure/service-health/resource-health-overview).

### Autolearn SNAT routes (preview)

You can configure Azure Firewall to autolearn both registered and private ranges every 30 minutes. For information, see [Azure Firewall SNAT private IP address ranges](snat-private-range.md#auto-learn-snat-routes-preview).

## Change tracking (preview)

The *Change tracking* feature provides detailed insights into changes made to Azure Firewall configurations, specifically within *Rule Collection Groups*. It uses [Azure Resource Graph (ARG)](../governance/resource-graph/overview.md) to enable efficient monitoring and analysis of changes, enhancing visibility, accountability, and troubleshooting.

For more information, see [Change tracking for Azure Firewall](monitor-firewall.md#change-tracking-preview).

## Customer provided public IP address support in secured hubs (preview)

Virtual WAN hub deployments can now associate customer tenant public IP addresses with Secured Hub Azure Firewall. The capability is available to new deployments of Secured Hub Firewalls (preview). 

For existing secured virtual WAN hubs, delete the hub firewall and redeploy a new Firewall during scheduled maintenance hours. You can use the Azure portal or Azure PowerShell to configure this.  

For more information, see [Customer provided public IP address support in secured hubs (preview)](secured-hub-customer-public-ip.md).

## Next steps

To learn more about Azure Firewall, see [What is Azure Firewall?](overview.md).
