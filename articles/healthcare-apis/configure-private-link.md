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

With Private Link, you can access your services securely from your virtual network as a first party service without having to go through a public Domain Name System (DNS). This article describes how to create, test, and manage your private endpoint for Azure Health Data Services.

>[!Note]
> Neither Private Link nor Azure Health Data Services can be moved from one resource group or subscription to another once Private Link is enabled. To make a move, delete the Private Link first, and then move Azure Health Data Services. Create a new Private Link after the move is complete. Next, assess potential security ramifications before deleting the Private Link.
>
>If you're exporting audit logs and metrics that are enabled, update the export setting through **Diagnostic Settings** from the portal.

## Prerequisites

Before you create a private endpoint, the following Azure resources must be created first:

- **Resource Group** – The Azure resource group that contains the virtual network and private endpoint.
- **Workspace** – The logical container for FHIR&reg; and DICOM&reg; service instances.
- **FHIR or DICOM service** – The Azure Health Data Services resource that you want to connect to over the private endpoint.  This is not required to create the private endpoint, but is required to test the private endpoint connectivity.
- To create a virtual network, you need to have an RBAC role for the resource group, such as **Owner**, **Contributor**, or **Network Contributor**. If you don't have permissions, contact your administrator.
- To create a private endpoint, you need to have an RBAC role for the workspace or the resource group where the workspace is located, such as **Owner**, **Contributor**, or **Healthcare APIs Contributor**. If you don't have permissions, contact your administrator.


## Create a virtual network and subnet

Virtual networks are created at the resource group level. When creating a virtual network, create a dedicated subnet for the private endpoint. This is required as part of the private endpoint creation process. You can create additional subnets for your client services that will connect to Azure Health Data Services over the private endpoint.

When creating the subnet for the private endpoint, make sure to not enable any service endpoints on that subnet. Service endpoints aren't compatible with private endpoints and can cause connectivity issues.

To create a virtual network and subnet, follow the steps below:

1. In the [Azure portal](https://portal.azure.com), select **Create a resource**.
1. Search for and select **Virtual Network**.
1. Select **Create**.
1. On the Basics tab, select the subscription and resource group that contains your workspace. 
1. Enter a name for the virtual network, and select a region.
1. Select the **IP Addresses** tab.  

:::image type="content" source="media/private-link/create-vnet-basics-tab.png" alt-text="Screenshot of the create virtual network Basics tab." lightbox="media/private-link/create-vnet-basics-tab.png":::

1. On the IP Addresses tab, enter an address space for the virtual network. 
1. Create a subnet for the private endpoint by selecting **+ Add subnet**. 

:::image type="content" source="media/private-link/create-vnet-ip-addresses-tab.png" alt-text="Screenshot of the create virtual network IP Addresses tab." lightbox="media/private-link/create-vnet-ip-addresses-tab.png":::

1. Enter a name and select the address range, starting address, and size for the subnet and select **Add** to add the subnet.

:::image type="content" source="media/private-link/create-vnet-add-subnet.png" alt-text="Screenshot of the create virtual network Add subnet tab." lightbox="media/private-link/create-vnet-add-subnet.png":::

1. Select **Review + create**, and then select **Create**.

:::image type="content" source="media/private-link/create-vnet-review-tab.png" alt-text="Screenshot of the create virtual network Review + create tab." lightbox="media/private-link/create-vnet-review-tab.png":::

## Create private endpoint

To create a private endpoint, a user with Role-based access control (RBAC) permissions on the workspace or the resource group where the workspace is located can use the Azure portal. Using the Azure portal is recommended as it automates the creation and configuration of the Private DNS Zone. For more information, see [Private Link Quick Start Guides](./../private-link/create-private-endpoint-portal.md).

A private endpoint is configured at the workspace level, and is automatically configured for all FHIR and DICOM services within the workspace. Services created after the private endpoint will be added to the private endpoint, however, a DNS record for the new service will need to be added to the Private DNS Zone. For more information, see [Private Link DNS configuration](#private-link-dns-configuration).

When you create a private endpoint for a workspace, you have the option to disable public network access. If you choose to disable public network access, all traffic to the workspace and its services must go through the private endpoint.

You can create a private endpoint from the Network foundation experience in the Azure portal, or by selecting the workspace and navigating to the **Networking** tab.

### Create a private endpoint from the Network foundation experience

Follow these steps to create a private endpoint from the Network foundation experience:

1. In the Azure portal, search for and select **Private endpoints** from the top search bar.
1. Select**+ Create** to create a new private endpoint.
1. On the Basics tab, select the subscription and resource group that contains your workspace. 
1. Enter a name for the private endpoint, and select a region. The region for the private endpoint must be the same as the region for the virtual network.
1. Select **Next: Resource >** to continue to the Resource tab.

:::image type="content" source="media/private-link/create-private-endpoint-basics-tab.png" alt-text="Screenshot showing image of the Azure portal Basics Tab." lightbox="media/private-link/create-private-endpoint-basics-tab.png":::

#### Resource configuration

There are two ways to assign a resource to the private endpoint. Auto Approval flow allows a user that has RBAC permissions on the workspace to create a private endpoint without a need for approval. Manual Approval flow allows a user without permissions on the workspace to request that owners of the workspace or resource group approve the private endpoint.



#### [Auto approval](#tab/auto-approval)

For auto approval, follow these steps:

1. For **Connection method**, select **Connect to an Azure resource in my directory**. 
1. For the resource type, search and select **Microsoft.HealthcareApis/workspaces** from the drop-down list. 
1. For the resource, select the workspace in the resource group. The target subresource, **healthcare workspace**, is automatically populated.
1. Select **Next: Virtual Network >** to continue to the **Virtual Network** tab.

:::image type="content" source="media/private-link/create-private-endpoint-auto-approval.png" alt-text="Screen image of the Auto Approval Resources tab." lightbox="media/private-link/create-private-endpoint-auto-approval.png":::

#### [Manual approval](#tab/manual-approval)

For manual approval, follow these steps:

1. For the **Connection method**, select the second option under Resource, **Connect to an Azure resource by resource ID or alias**. 
1. For the resource ID, enter `subscriptions/{subscriptionid}/resourceGroups/{resourcegroupname}/providers/Microsoft.HealthcareApis/workspaces/{workspacename}`. 
1. For the Target subresource, enter **healthcareworkspace**.
1. enter a message for the approver in the **Message for approver** field to provide context for the approval request.
1. Select **Next: Virtual Network >** to continue to the **Virtual Network** tab.

:::image type="content" source="media/private-link/create-private-endpoint-manual-approval.png" alt-text="Screen image of the Manual Approval Resources tab." lightbox="media/private-link/create-private-endpoint-manual-approval.png":::

When using manual approval, you can't integrate with a private DNS zone as part of the private endpoint creation process. To use Azure You will need to manually create a private DNS zone and link it to your virtual network. For more information, see [Private Link DNS configuration](#private-link-dns-configuration).

> [!NOTE]
> When an approved private endpoint is created for Azure Health Data Services, public traffic to it's automatically disabled. 
---


#### Virtual network configuration

1. For the **Virtual network**, select the virtual network that you created for the private endpoint.
1. For the **Subnet**, select the subnet that you created for the private endpoint.
1. If you want to setup Network Security Group (NSG) rules or route tables to restrict the traffic to the private endpoint, you can select **edit** the **Network policy for private endpoints**. 
1. For **Private IP configuration**, you can choose to have an IP address automatically assigned from the subnet, or you can specify a static IP address from the subnet.
1. Select **Next: DNS >** to continue to the **DNS** tab.

:::image type="content" source="media/private-link/create-private-endpoint-vnet-tab.png" alt-text="Screenshot showing image of the Azure portal Virtual Network tab." lightbox="media/private-link/create-private-endpoint-vnet-tab.png":::

#### DNS configuration

If you used the auto approval method, you can integrate with a private DNS zone as part of the private endpoint creation process. If you used the manual approval method, you will need to manually create a private DNS zone and link it to your virtual network. For more information, see [Private Link DNS configuration](#private-link-dns-configuration).

You can choose to have the private endpoint automatically integrate with a private DNS zone, or you can choose to not integrate with a private DNS zone. If you choose to integrate with a private DNS zone, 
two private DNS zones will be created, one for the workspace and FHIR services and one for DICOM services. The private DNS zones are automatically linked to the virtual network that you selected for the private endpoint.

:::image type="content" source="media/private-link/create-private-endpoint-dns-tab.png" alt-text="Screenshot showing image of the Azure portal DNS tab." lightbox="media/private-link/create-private-endpoint-dns-tab.png":::

Go to the **Review + create** tab and review your configuration. If everything looks correct, select **Create** to create the private endpoint.

### Create the private endpoint from the workspace Networking tab

To create a private endpoint from the workspace Networking tab, follow these steps:
1. In the Azure portal, navigate to your workspace.
1. Select **Networking** from the left-hand menu, and then select **+ Private endpoint**.
1. Enter the resource group where your private endpoint will be created, and the name for your private endpoint. The region is automatically populated based on the region of your workspace. Ensure that the region for the private endpoint is the same as the region for the virtual network.
1. Select your virtual network and subnet for the private endpoint. Remember to not enable any service endpoints on the subnet you select for the private endpoint.
1. For **Integrate with private DNS zone**, you can choose to have the private endpoint automatically integrate with a private DNS zone, or you can choose to not integrate with a private DNS zone. If you choose to integrate with a private DNS zone, a private DNS zones will be created for the workspace and FHIR services and one for DICOM services. The private DNS zones are automatically linked to the virtual network that you selected for the private endpoint.
1. Select **OK** to create the private endpoint.

## Private Link DNS configuration

After the deployment is complete, select the private endpoint resource in the resource group. Open **DNS configuration** from the settings menu. You can see the IP address assignments for each service connected to the private endpoint. The private DNS zone configurations and the DNS records are listed with the IP addresses. You can find the DNS records and private IP addresses for the workspace, and FHIR and DICOM services.

:::image type="content" source="media/private-link/private-link-dns-configuration.png" alt-text="Screenshot showing image of the Azure portal DNS Configuration.":::

When you add a new service to the workspace, a new DNS record needs to be added to the Private DNS Zone for that service. To add a new DNS record:

1. Select the private DNS zone and then select **Recordsets** and **+ Add**. 
1. For the **Name**, enter the name of the service as it appears in the audience for fhir or the service URL for DICOM.  For example, `workspace-fhir-2.fhir` or `workspace-dicom-2.dicom`.
1. For the record type, select **A**. 
1. Set the **TTL** or leave as default. Record sets with a lower TTL will have more frequent DNS lookups, which can help with faster failover in the event of an issue, but can also increase latency and DNS query costs. Consider your application's needs when setting the TTL value. 
1. For the **IP address**, enter the private IP address for that service from the DNS configuration page.
1. Select **Add** to add the record set.

## Private Link Mapping

After the deployment is complete, browse to the your workspace and select that is created as part of the deployment. You should see two private DNS zone records and one for each service. If you have more FHIR and DICOM services in the workspace, more DNS zone records are created for them.

:::image type="content" source="media/private-link/private-link-fhir-map.png" alt-text="Screenshot showing image of Private Link FHIR Mapping.":::

Select **Virtual network links** from the **Settings**. Notice that the FHIR service is linked to the virtual network. 
Make sure only a single VNET is associated with the DNS zone. If you need to support multiple VNETs, you must create separate DNS zones in different resource groups. During the setup confirm that the Private Endpoint and Private DNS Zone aren't shared across multiple VNETs, as this is a common misconfiguration that can lead to IP resolution issues and access failures leading to HTTP 403 errors on the service.

:::image type="content" source="media/private-link/private-link-vnet-link-fhir.png" alt-text="Screenshot showing image of Private Link virtual network Link FHIR.":::

Similarly, you can see the private link mapping for the DICOM service.

:::image type="content" source="media/private-link/private-link-dicom-map.png" alt-text="Screenshot showing image of Private Link DICOM Mapping.":::

Also, you can see the DICOM service is linked to the virtual network.

:::image type="content" source="media/private-link/private-link-vnet-link-dicom.png" alt-text="Screenshot showing image of Private Link virtual network Link DICOM.":::

## Test private endpoint

To verify that your service isn’t receiving public traffic after disabling public network access, select the `/metadata` endpoint for your FHIR service, or the /health/check endpoint of the DICOM service, and you'll receive the message 403 Forbidden. 

It can take up to 5 minutes after updating the public network access flag before public traffic is blocked.

> [!IMPORTANT]
> Every time a new service gets added into the Private Link enabled workspace, wait for the provisioning to complete. Refresh the private endpoint if DNS A records aren't getting updated for the newly added services in the workspace. If DNS A records aren't updated in your private DNS zone, requests to a newly added services won't go over Private Link. 

To ensure your Private Endpoint can send traffic to your server:

1. Create a virtual machine (VM) that is connected to the virtual network and subnet your Private Endpoint is configured on. To ensure your traffic from the VM is only using the private network, disable the outbound internet traffic using the network security group (NSG) rule.
2. Remote Desktop Protocols (RDP) into the VM.
3. Access your FHIR server’s `/metadata` endpoint from the VM. You should receive the capability statement as a response.

## FAQ

### 1. FHIR service configured with Private endpoints is missing their private link DNS entries, what should I do?
FHIR services configured with a private endpoint that are missing private link DNS entries will resolve through the public CNAME path instead of resolving to the private IP address via *.private link.fhir.azurehealthcareapis.com. This is a known issue with the FHIR service that can intermittently occur during provisioning and may prevent the correct configuration of the private link DNS entries.
Due to this issue, services may be unreachable from the virtual network (VNet), which can result in connectivity failures for applications relying on private network access.

To mitigate this issue, remove and readd the private endpoint connection to the Azure Health Data Services (AHDS) Workspace. This action triggers a new provisioning cycle that correctly configures the private link DNS entries.

Follow the steps below to resolve the issue:
1. Navigate to the AHDS Workspace in the Azure portal.
2. Select Networking → Private endpoint connections.
3. Remove the existing Private Endpoint.
4. Re-create the Private Endpoint using the same configuration.
5. Verify that DNS resolution returns the private IP address.

### 2. From logs, requests failing with HTTP 403 are not due to bad tokens but instead they're rejected by Private Links, as their origin isn't allowed to access the FHIR service.

Validate the below points:

-  Check if the request origin of those requests is part of the same Virtual Network (VNET) where the FHIR service is.  

-  Check if the Private Endpoint and/or Private DNS zone are shared with multiple VNETs at the same time. This is a known misconfiguration that can cause turbulence on the IP resolution and result in requests being rejected.

## Related articles

- [Create a private endpoint using the Azure portal](./../private-link/create-private-endpoint-portal.md)
- [Private Link Documentation](./../private-link/index.yml).

[!INCLUDE [FHIR and DICOM trademark statement](./includes/healthcare-apis-fhir-dicom-trademark.md)]
