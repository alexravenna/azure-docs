---
title: Upgrade ingress controller with Helm
description: This article provides information on how to upgrade an Application Gateway Ingress Controller by using Helm.
services: application-gateway
author: mbender-ms
ms.service: azure-application-gateway
ms.topic: how-to
ms.date: 4/4/2025
ms.author: mbender
# Customer intent: "As a Kubernetes administrator, I want to upgrade the Application Gateway Ingress Controller using Helm, so that I can ensure my ingress solution is running the latest features and improvements."
---

# Upgrade AGIC by using Helm

You can upgrade the Azure Application Gateway Ingress Controller (AGIC) for Kubernetes by using a Helm repository hosted on Azure Storage.

> [!TIP]
> Consider [Application Gateway for Containers](for-containers/overview.md) for your Kubernetes ingress solution. For more information, see [Quickstart: Deploy Application Gateway for Containers ALB Controller](for-containers/quickstart-deploy-application-gateway-for-containers-alb-controller.md).

## Upgrade

1. View the currently installed Helm charts:

    ```bash
    helm list
    ```

    Sample response:

    ```output
    NAME            REVISION        UPDATED                         STATUS  CHART                   APP VERSION     NAMESPACE
    odd-billygoat   22              Fri Jun 21 15:56:06 2019        FAILED  ingress-azure-0.7.0-rc1 0.7.0-rc1       default
    ```

    The Helm chart installation from the preceding sample response is named `odd-billygoat`. This article uses that name for the commands. Your actual deployment name will be different.

1. Upgrade the Helm deployment to a new version:

    ```bash
    helm upgrade \
        odd-billygoat \
        oci://mcr.microsoft.com/azure-application-gateway/charts/ingress-azure \
        --version 1.8.1
    ```

## Roll back

If the Helm deployment fails, you can roll back to a previous release:

1. Get the number of the last known healthy release:

    ```bash
    helm history odd-billygoat
    ```

    Sample output:

    ```output
    REVISION        UPDATED                         STATUS          CHART                   DESCRIPTION
    1               Mon Jun 17 13:49:42 2019        DEPLOYED        ingress-azure-0.6.0     Install complete
    2               Fri Jun 21 15:56:06 2019        FAILED          ingress-azure-xx        xxxx
    ```

    Based on the sample output of the `helm history` command, the last successful deployment of the `odd-billygoat` example was revision `1`.

1. Roll back to the last successful revision:

    ```bash
    helm rollback odd-billygoat 1
    ```

## Related content

- [Application Gateway for Containers](for-containers/overview.md)
