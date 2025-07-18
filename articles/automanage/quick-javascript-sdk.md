---
title: Azure Quickstart SDK for JavaScript
description: Create configuration profile assignments using the JavaScript SDK for Automanage.
author: andrsmith
ms.service: azure-automanage
ms.custom: devx-track-js
ms.topic: quickstart
ms.date: 08/24/2022
ms.author: andrsmith
# Customer intent: As a cloud developer, I want to utilize the JavaScript SDK to enable Azure Automanage for my virtual machines, so that I can apply best practices efficiently to optimize management and compliance across my resources.
---

# Quickstart: Enable Azure Automanage for virtual machines using JavaScript

> [!CAUTION]
> On September 30, 2027, the Azure Automanage Best Practices service will be retired. As a result, attempting to create a new configuration profile or onboarding a new subscription to the service will result in an error. Learn more [here](https://aka.ms/automanagemigration/) about how to migrate to Azure Policy before that date. 

> [!CAUTION]
> Starting February 1st 2025, Azure Automanage will begin rolling out changes to halt support and enforcement for all services dependent on the deprecated Microsoft Monitoring Agent (MMA). To continue using Change Tracking and Management, VM Insights, Update Management, and Azure Automation, [migrate to the new Azure Monitor Agent (AMA)](https://aka.ms/mma-to-ama/).

Azure Automanage allows users to seamlessly apply Azure best practices to their virtual machines. This quickstart guide will help you apply a Best Practices Configuration profile to an existing virtual machine using the [azure-sdk-for-js repo](https://github.com/Azure/azure-sdk-for-js).

## Prerequisites 

- An active [Azure Subscription](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/)
- An existing [Virtual Machine](/azure/virtual-machines/windows/quick-create-portal)

> [!NOTE]
> Free trial accounts do not have access to the virtual machines used in this tutorial. Please upgrade to a Pay-As-You-Go subscription.

> [!IMPORTANT]
> You need to have the **Contributor** role on the resource group containing your VMs to enable Automanage. If you are enabling Automanage for the first time on a subscription, you need the following permissions: **Owner** role or **Contributor** along with **User Access Administrator** roles on your subscription.

## Install required packages 

For this demo, both the **Azure Identity** and **Azure Automanage** packages are required.

```
npm install @azure/arm-automanage
npm install @azure/identity
```

## Import packages 

Import the **Azure Identity** and **Azure Automanage** packages into the script: 

```javascript
const { AutomanageClient } = require("@azure/arm-automanage");
const { DefaultAzureCredential } = require("@azure/identity");
```

## Authenticate to Azure and create an Automanage client

Use the **Azure Identity** package to authenticate to Azure and then create an Automanage Client:

```javascript 
const credential = new DefaultAzureCredential();
const client = new AutomanageClient(credential, "<subscription ID>");
```

## Enable best practices configuration profile to an existing virtual machine

```javascript 
let assignment = {
    "properties": {
        "configurationProfile": "/providers/Microsoft.Automanage/bestPractices/AzureBestPracticesProduction"
    }
}

// assignment name must be named "default"
await client.configurationProfileAssignments.createOrUpdate("default", "resourceGroupName", "vmName", assignment);
```

## Next steps

> [!div class="nextstepaction"]
Learn how to conduct more operations with the JavaScript Automanage Client by visiting the [azure-sdk-for-js repo](https://github.com/Azure/azure-sdk-for-js/tree/main/sdk/automanage/arm-automanage).
