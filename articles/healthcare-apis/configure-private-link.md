---
title: Configure Private Link for Azure Health Data Services
description: Learn how to set up Private Link for secure access to Azure Health Data Services.
services: healthcare-apis
author: EXPEkesheth
ms.service: azure-health-data-services
ms.subservice: fhir
ms.topic: tutorial
ms.date: 06/02/2025
ms.author: kesheth
ms.custom: sfi-image-nochange
---

# Configure Private Link for Azure Health Data Services

Azure Private Link enables you to access Azure Health Data Services over a private endpoint in your virtual network. A private endpoint is a network interface that connects you privately and securely using a private IP address from your virtual network. This interface is backed by Azure Private Link. 

By using Private Link, you can access your services securely from your virtual network as a first-party service without going through a public Domain Name System (DNS). This article describes how to create, test, and manage your private endpoint for Azure Health Data Services.

>[!NOTE]
> You can't move Private Link or Azure Health Data Services from one resource group or subscription to another once Private Link is enabled. To make a move, delete the Private Link first, and then move Azure Health Data Services. Create a new Private Link after the move is complete. Next, assess potential security ramifications before deleting the Private Link.
>
>If you're exporting audit logs and metrics that are enabled, update the export setting through **Diagnostic Settings** from the portal.

## Prerequisites

Before you create a private endpoint, create the following Azure resources:

- **Resource Group** – The Azure resource group that contains the virtual network and private endpoint.
- **Workspace** – The logical container for FHIR&reg; and DICOM&reg; service instances.
- **FHIR or DICOM service** – The Azure Health Data Services resource that you want to connect to over the private endpoint.  This resource isn't required to create the private endpoint, but is required to test the private endpoint connectivity.
- To create a virtual network, you need to have an RBAC role for the resource group, such as **Owner**, **Contributor**, or **Network Contributor**. If you don't have permissions, contact your administrator.
- To create a private endpoint, you need to have an RBAC role for the workspace or the resource group where the workspace is located, such as **Owner**, **Contributor**, or **Healthcare APIs Contributor**. If you don't have permissions, contact your administrator.


## Create a virtual network and subnet

Create virtual networks at the resource group level. When you create a virtual network, create a dedicated subnet for the private endpoint. This subnet is required as part of the private endpoint creation process. You can create extra subnets for your client services that connect to Azure Health Data Services over the private endpoint.

When you create the subnet for the private endpoint, don't enable any service endpoints on that subnet. Service endpoints aren't compatible with private endpoints and can cause connectivity problems.

To create a virtual network and subnet, follow these steps:

1. In the [Azure portal](https://portal.azure.com), select **Create a resource**.
1. Search for and select **Virtual Network**.
1. Select **Create**.
1. On the **Basics** tab, select the subscription and resource group that contains your workspace. 
1. Enter a name for the virtual network, and select a region.
1. Select the **IP Addresses** tab.  

    :::image type="content" source="media/private-link/create-vnet-basics-tab.png" alt-text="Screenshot of the create virtual network Basics tab." lightbox="media/private-link/create-vnet-basics-tab.png":::

1. On the **IP Addresses** tab, enter an address space for the virtual network. 
1. Create a subnet for the private endpoint by selecting **+ Add subnet**. 

    :::image type="content" source="media/private-link/create-vnet-ip-addresses-tab.png" alt-text="Screenshot of the create virtual network IP Addresses tab." lightbox="media/private-link/create-vnet-ip-addresses-tab.png":::

1. Enter a name and select the address range, starting address, and size for the subnet. Select **Add** to add the subnet.

    :::image type="content" source="media/private-link/create-vnet-add-subnet.png" alt-text="Screenshot of the create virtual network Add subnet tab." lightbox="media/private-link/create-vnet-add-subnet.png":::

1. Select **Review + create**, and then select **Create**.

    :::image type="content" source="media/private-link/create-vnet-review-tab.png" alt-text="Screenshot of the create virtual network Review + create tab." lightbox="media/private-link/create-vnet-review-tab.png":::

## Create private endpoint

To create a private endpoint, use the Azure portal as a user with Role-based access control (RBAC) permissions on the workspace or the resource group where the workspace is located. Use the Azure portal because it automates the creation and configuration of the Private DNS Zone. For more information, see [Private Link Quick Start Guides](./../private-link/create-private-endpoint-portal.md).

You configure a private endpoint at the workspace level. The private endpoint automatically applies to all FHIR and DICOM services within the workspace. If you create services after the private endpoint, add those services to the private endpoint. However, you need to add a DNS record for the new service to the Private DNS Zone. For more information, see [Private Link DNS configuration](#private-link-dns-configuration).

When you create a private endpoint for a workspace, you can disable public network access. If you disable public network access, all traffic to the workspace and its services must go through the private endpoint.

You can create a private endpoint from the Network foundation experience in the Azure portal, or by selecting the workspace and navigating to the **Networking** tab.

### Create a private endpoint from the Network foundation experience

Follow these steps to create a private endpoint from the Network foundation experience:

1. In the Azure portal, search for and select **Private endpoints** from the top search bar.
1. Select **+ Create** to create a new private endpoint.
1. On the **Basics** tab, select the subscription and resource group that contains your workspace. 
1. Enter a name for the private endpoint, and select a region. The region for the private endpoint must be the same as the region for the virtual network.
1. Select **Next: Resource >** to continue to the **Resource** tab.

:::image type="content" source="media/private-link/create-private-endpoint-basics-tab.png" alt-text="Screenshot showing image of the Azure portal Basics Tab." lightbox="media/private-link/create-private-endpoint-basics-tab.png":::

#### Resource configuration

Assign a resource to the private endpoint in one of two ways. The auto approval flow enables a user with RBAC permissions on the workspace to create a private endpoint without needing approval. The manual approval flow enables a user without permissions on the workspace to request that owners of the workspace or resource group approve the private endpoint.



#### [Auto approval](#tab/auto-approval)

For auto approval, follow these steps:

1. For **Connection method**, select **Connect to an Azure resource in my directory**. 
1. For the resource type, search for and select **Microsoft.HealthcareApis/workspaces** from the drop-down list. 
1. For the resource, select the workspace in the resource group. The **Target sub-resource** is automatically populated with  **healthcareworkspace**.
1. Select **Next: Virtual Network >** to continue to the **Virtual Network** tab.

:::image type="content" source="media/private-link/create-private-endpoint-auto-approval.png" alt-text="Screen image of the Auto Approval Resources tab." lightbox="media/private-link/create-private-endpoint-auto-approval.png":::

#### [Manual approval](#tab/manual-approval)

For manual approval, follow these steps:

1. For the **Connection method**, select the second option under Resource, **Connect to an Azure resource by resource ID or alias**. 
1. For the resource ID, enter `subscriptions/{subscriptionid}/resourceGroups/{resourcegroupname}/providers/Microsoft.HealthcareApis/workspaces/{workspacename}`. 
1. For the **Target sub-resource**, enter `healthcareworkspace`.
1. Enter a message for the approver in the **Message for approver** field to provide context for the approval request.
1. Select **Next: Virtual Network >** to continue to the **Virtual Network** tab.

:::image type="content" source="media/private-link/create-private-endpoint-manual-approval.png" alt-text="Screen image of the Manual Approval Resources tab." lightbox="media/private-link/create-private-endpoint-manual-approval.png":::

When you use manual approval, you can't integrate with a private DNS zone as part of the private endpoint creation process. To use Azure Private DNS, you need to manually create a private DNS zone and link it to your virtual network. For more information, see [Private Link DNS configuration](#private-link-dns-configuration).

> [!NOTE]
> When you create an approved private endpoint for Azure Health Data Services, it automatically disables public traffic. 

---


#### Virtual network configuration

1. For the **Virtual network**, select the virtual network that you created for the private endpoint.
1. For the **Subnet**, select the subnet that you created for the private endpoint.
1. To set up Network Security Group (NSG) rules or route tables to restrict the traffic to the private endpoint, select **edit** the **Network policy for private endpoints**. 
1. For **Private IP configuration**, choose to have an IP address automatically assigned from the subnet, or specify a static IP address from the subnet.
1. Select **Next: DNS >** to continue to the **DNS** tab.

:::image type="content" source="media/private-link/create-private-endpoint-vnet-tab.png" alt-text="Screenshot showing image of the Azure portal Virtual Network tab." lightbox="media/private-link/create-private-endpoint-vnet-tab.png":::

#### DNS configuration

If you use the auto approval method, you can integrate with Azure Private DNS zones as part of the private endpoint creation process. If you use the manual approval method and want to integrate with Azure Private DNS zones, you need to manually create a private DNS zone and link it to your virtual network. For more information, see [Private Link DNS configuration](#private-link-dns-configuration).

You can choose whether to have the private endpoint automatically integrate with a private DNS zone. If you choose to integrate with a private DNS zone, two private DNS zones are created, one for the workspace and FHIR services and one for DICOM services. The private DNS zones are automatically linked to the virtual network that you selected for the private endpoint.

:::image type="content" source="media/private-link/create-private-endpoint-dns-tab.png" alt-text="Screenshot showing image of the Azure portal DNS tab." lightbox="media/private-link/create-private-endpoint-dns-tab.png":::

Go to the **Review + create** tab and review your configuration. If everything looks correct, select **Create** to create the private endpoint.

### Create the private endpoint from the workspace Networking tab

To create a private endpoint from the workspace Networking tab, follow these steps:
1. In the Azure portal, go to your workspace.
1. Select **Networking** from the left-hand menu, and then select **+ Private endpoint**.
1. Enter the resource group where you want to create your private endpoint, and the name for your private endpoint. The region is automatically populated based on the region of your workspace. Make sure that the region for the private endpoint is the same as the region for the virtual network.
1. Select your virtual network and subnet for the private endpoint. Don't enable any service endpoints on the subnet you select for the private endpoint.
1. For **Integrate with private DNS zone**, choose whether to have the private endpoint automatically integrate with an Azure Private DNS zone. When you choose to integrate with a private DNS zone, the portal creates a private DNS zone that is automatically linked to the virtual network that you selected for the private endpoint.
1. Select **OK** to create the private endpoint.

## Private Link DNS configuration

After the deployment finishes, select the private endpoint resource in the resource group. Open **Settings** > **DNS configuration**. You see the IP address assignments for each service connected to the private endpoint. If you chose to integrate your private endpoint with Azure Private DNS zones, the private DNS zone configurations and the DNS records for each service are listed.

:::image type="content" source="media/private-link/private-link-dns-configuration.png" alt-text="Screenshot showing private endpoint DNS Configuration.":::

You can add new Azure Private DNS zones and DNS records to support additional services or custom domain names. To add a new Azure Private DNS zone, enter **Private DNS zones** in the portal search box and select **+ Create**. From here, follow the prompts to create the new DNS zone and link it to your virtual network.

When you add new services to the workspace, add new DNS records to your DNS server or Azure Private DNS Zone for those services. 

To add a new DNS record to an Azure Private DNS Zone, follow these steps:

1. Select the private DNS zone and then select **Recordsets** and **+ Add**. 
1. For the **Name**, enter the name of the service as it appears in the audience for FHIR or the service URL for DICOM.  For example, `workspace-fhir-2.fhir` or `workspace-dicom-2.dicom`.
1. For the record type, select **A**. 
1. Set the **TTL** or leave as default. Record sets with a lower TTL have more frequent DNS lookups, which can help with faster failover in the event of an issue, but can also increase latency and DNS query costs. Consider your application's needs when setting the TTL value. 
1. For the **IP address**, enter the private IP address for that service from the DNS configuration page.
1. Select **Add** to add the record set.

## Private Link mapping

After the deployment finishes, browse to the workspace and select the workspace created as part of the deployment. You see two private DNS zone records, one for each service. If you add more FHIR and DICOM services in the workspace, you create more DNS zone records for them.

:::image type="content" source="media/private-link/private-link-fhir-map.png" alt-text="Screenshot showing image of Private Link FHIR Mapping.":::

Select **Virtual network links** from the **Settings**. Notice that the FHIR service is linked to the virtual network. 
Make sure you associate only a single virtual network with the DNS zone. If you need to support multiple virtual networks, you must create separate DNS zones in different resource groups. During the setup, confirm that the Private Endpoint and Private DNS Zone aren't shared across multiple virtual networks. This common misconfiguration can lead to IP resolution problems and access failures that result in HTTP 403 errors on the service.

:::image type="content" source="media/private-link/private-link-vnet-link-fhir.png" alt-text="Screenshot showing image of Private Link virtual network Link FHIR.":::

Similarly, you can see the private link mapping for the DICOM service.

:::image type="content" source="media/private-link/private-link-dicom-map.png" alt-text="Screenshot showing image of Private Link DICOM Mapping.":::

Also, you see the DICOM service is linked to the virtual network.

:::image type="content" source="media/private-link/private-link-vnet-link-dicom.png" alt-text="Screenshot showing image of Private Link virtual network Link DICOM.":::

## Test private endpoint

To verify that your service isn't receiving public traffic after disabling public network access, select the `/metadata` endpoint for your FHIR service, or the `/health/check` endpoint of the DICOM service, and you receive the message 403 Forbidden. 

It can take up to five minutes after updating the public network access flag before public traffic is blocked.

> [!IMPORTANT]
> Every time you add a new service into the Private Link enabled workspace, wait for the provisioning to complete. Refresh the private endpoint if DNS A records aren't getting updated for the newly added services in the workspace. If DNS A records aren't updated in your private DNS zone, requests to a newly added services won't go over Private Link. 

To ensure your Private Endpoint can send traffic to your server:

1. Create a virtual machine (VM) that is connected to the virtual network and subnet your Private Endpoint is configured on. To ensure your traffic from the VM only uses the private network, disable the outbound internet traffic by using the network security group (NSG) rule.
1. Use Remote Desktop Protocol (RDP) to connect to the VM.
1. Access your FHIR server’s `/metadata` endpoint from the VM. You should receive the capability statement as a response.

## FAQ

### 1. FHIR service configured with private endpoints is missing its private link DNS entries, what should I do?
If a FHIR service configured with a private endpoint is missing private link DNS entries, it resolves through the public CNAME path instead of resolving to the private IP address via `*.private link.fhir.azurehealthcareapis.com`. This problem can intermittently occur during provisioning and might prevent the correct configuration of the private link DNS entries.
Due to this problem, services might be unreachable from the virtual network (VNet), which can result in connectivity failures for applications relying on private network access.

To mitigate this problem, remove and re-add the private endpoint connection to the Azure Health Data Services (AHDS) Workspace. This action triggers a new provisioning cycle that correctly configures the private link DNS entries.

Follow the steps in the following list to resolve the problem:
1. Go to the AHDS Workspace in the Azure portal.
1. Select **Networking** > **Private endpoint connections**.
1. Remove the existing private endpoint.
1. Re-create the private endpoint by using the same configuration.
1. Verify that DNS resolution returns the private IP address.

### 2. Logs show that requests fail with HTTP 403. The failures aren't due to bad tokens but instead Private Links rejects the requests because their origin isn't allowed to access the FHIR service.

Validate the following points:

-  Check if the request origin of those requests is part of the same virtual network (VNET) where the FHIR service is.  

-  Check if the private endpoint and private DNS zone are shared with multiple VNETs at the same time. This known misconfiguration can cause turbulence on the IP resolution and result in requests being rejected.

## Related articles

- [Create a private endpoint using the Azure portal](./../private-link/create-private-endpoint-portal.md)
- [Private Link Documentation](./../private-link/index.yml).

[!INCLUDE [FHIR and DICOM trademark statement](./includes/healthcare-apis-fhir-dicom-trademark.md)]
