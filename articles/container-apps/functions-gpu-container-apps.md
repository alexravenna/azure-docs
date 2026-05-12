---
title: Use GPUs with Azure Functions on Azure Container Apps
description: Learn how to use serverless GPUs with Azure Functions on Azure Container Apps for compute-intensive AI and processing workloads.
author: deepganguly
ms.author: deepganguly
ms.service: azure-container-apps
ms.date: 05/12/2026
ms.topic: how-to
ms.custom:
  - build-2025
  - linux-related-content
# Customer intent: As a developer, I want to use GPUs with Azure Functions on Azure Container Apps so that I can run compute-intensive AI workloads in an event-driven model.
---

# Use GPUs with Azure Functions on Azure Container Apps

Azure Functions on Azure Container Apps supports GPU-enabled hosting, allowing you to run compute-intensive workloads like AI inference, image processing, and machine learning directly from your event-driven function apps. By combining the serverless programming model of Azure Functions with GPU compute on Container Apps, you can process GPU workloads that automatically scale based on demand.

## Overview

When you host Azure Functions on [Azure Container Apps](overview.md), you can use [serverless GPUs](gpu-serverless-overview.md) to access NVIDIA A100 and T4 GPU resources on demand. Serverless GPUs scale to zero when idle, so you only pay for GPU compute when your functions are actively processing requests.

GPU support for Functions on Container Apps is available through Consumption GPU workload profiles in a workload profiles environment. You package your function code and GPU dependencies (such as CUDA libraries, AI models, or inference frameworks) into a custom container image, then deploy it to a Container Apps environment with a GPU workload profile.

## Prerequisites

Before you begin, make sure you have the following requirements:

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account).
- [GPU quota approved](gpu-serverless-overview.md#request-serverless-gpu-quota) for your subscription. Customers with enterprise agreements and pay-as-you-go subscriptions have A100 and T4 quota enabled by default.
- [Azure CLI](/cli/azure/install-azure-cli) version 2.62.0 or later.
- [Azure Functions Core Tools](../azure-functions/functions-run-local.md) version 4.x.

## Container image requirements for GPU

When running Azure Functions with GPU on Container Apps, you must provide a custom container image that includes the Functions runtime, your GPU libraries, and your application code. Understanding the image requirements helps you build containers that work correctly with serverless GPUs.

### Base image

Your container image must include the Azure Functions runtime. Use one of the official [Azure Functions base images](https://mcr.microsoft.com/catalog?search=functions) as your starting point:

| Language | Base image |
|---|---|
| Python | `mcr.microsoft.com/azure-functions/python:4-python3.11` |
| Node.js | `mcr.microsoft.com/azure-functions/node:4-node20` |
| .NET | `mcr.microsoft.com/azure-functions/dotnet-isolated:4-dotnet-isolated8.0` |
| Java | `mcr.microsoft.com/azure-functions/java:4-java17` |
| Custom handler | `mcr.microsoft.com/azure-functions/base:4` |

These base images include the Functions host runtime. The [quickstart image](https://mcr.microsoft.com/en-us/artifact/mar/k8se/gpu-quickstart) (`mcr.microsoft.com/k8se/gpu-quickstart:latest`) is a prebuilt sample for testing GPU functionality, but it doesn't include the Functions runtime — use it only for validating GPU access in your environment.

> [!TIP]
> When you create a project with `func init --docker`, the generated Dockerfile already uses the correct base image for your chosen language runtime.

### CUDA and GPU libraries

The Azure Container Apps serverless GPU platform provides both the NVIDIA driver and a [platform-provided CUDA runtime](gpu-serverless-overview.md#gpu-software-stack) (currently CUDA 12.x). If your application can use the platform-provided CUDA version, no extra CUDA setup is needed in your container. However, AI/ML frameworks and additional libraries like cuDNN must be included in your container image.

You have two options for CUDA:

**Option 1: Use the platform-provided CUDA runtime with GPU frameworks** (recommended)

Most AI/ML frameworks like PyTorch and TensorFlow bundle their own CUDA runtime. Install them with the appropriate CUDA variant:

```dockerfile
FROM mcr.microsoft.com/azure-functions/python:4-python3.11

# PyTorch includes CUDA runtime
RUN pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121

COPY . /home/site/wwwroot
ENV AzureWebJobsScriptRoot=/home/site/wwwroot
```

**Option 2: Pin a specific CUDA version with a multi-stage build**

If you need a specific CUDA version different from the platform-provided one, or need direct CUDA access without a high-level framework:

```dockerfile
# Stage 1: CUDA runtime
FROM nvidia/cuda:12.1.0-runtime-ubuntu22.04 AS cuda-base

# Stage 2: Functions image with CUDA libraries
FROM mcr.microsoft.com/azure-functions/python:4-python3.11

# Copy CUDA libraries from the NVIDIA base image
COPY --from=cuda-base /usr/local/cuda /usr/local/cuda
ENV PATH="/usr/local/cuda/bin:${PATH}"
ENV LD_LIBRARY_PATH="/usr/local/cuda/lib64:${LD_LIBRARY_PATH}"

COPY requirements.txt /
RUN pip install -r /requirements.txt

COPY . /home/site/wwwroot
ENV AzureWebJobsScriptRoot=/home/site/wwwroot
```

### GPU software stack compatibility

The platform provides the NVIDIA driver and a CUDA runtime. Your container can rely on the platform-provided CUDA or ship its own. Check the current supported versions in the [GPU software stack](gpu-serverless-overview.md#gpu-software-stack) documentation.

| Component | Provided by |
|---|---|
| NVIDIA GPU driver | Platform (Azure Container Apps host) |
| CUDA runtime | Platform (default) or your container image (optional override) |
| cuDNN, NCCL, TensorRT | Your container image |
| AI/ML frameworks (PyTorch, TensorFlow, etc.) | Your container image |

> [!IMPORTANT]
> If you ship your own CUDA runtime, it must be compatible with the NVIDIA driver version on the platform. Generally, the NVIDIA driver is backward-compatible with older CUDA versions. Check the [NVIDIA CUDA compatibility matrix](https://docs.nvidia.com/deploy/cuda-compatibility/) for details. If you rely on the platform-provided CUDA runtime, verify your application works with the [current platform versions](gpu-serverless-overview.md#gpu-software-stack).

### Image size considerations

GPU container images are typically large (5-15 GB) due to CUDA libraries and model files. To reduce image pull times and cold start latency:

- Use multi-stage Docker builds to minimize the final image size.
- Store large model files in [Azure Storage mounts](cold-start.md#manage-large-downloads) instead of bundling them in the image.
- Enable [artifact streaming](/azure/container-registry/container-registry-artifact-streaming) on your Azure Container Registry (Premium SKU) for faster image pulls.
- Use `.dockerignore` to exclude unnecessary files from the build context.

### Sample Dockerfiles and templates

For ready-to-use examples, see the following resources:

- [Azure Functions on Container Apps GPU sample](https://github.com/Azure-Samples/function-on-aca-gpu) — Complete sample with Dockerfile, function code, and deployment configuration.
- [Azure Container Apps GPU templates](https://github.com/microsoft/azure-container-apps/tree/main/templates/azml-app) — Templates for deploying models to serverless GPUs.
- [Azure Functions base images](https://mcr.microsoft.com/catalog?search=functions) — Official Functions runtime images for all supported languages.

## Create a GPU-enabled function app

### Step 1: Create a Functions project with Docker support

Create a new Functions project with a Dockerfile:

```bash
func init MyGpuFunctionApp --worker-runtime python --docker
cd MyGpuFunctionApp
```

### Step 2: Add a function

Create an HTTP-triggered function:

```bash
func new --name GpuProcess --template "HTTP trigger"
```

### Step 3: Update the Dockerfile for GPU support

Replace the contents of your `Dockerfile` with a GPU-enabled base image. The following example uses PyTorch for AI inference:

```dockerfile
FROM mcr.microsoft.com/azure-functions/python:4-python3.11

# Install GPU dependencies
RUN pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121
RUN pip install -r requirements.txt

ENV AzureWebJobsScriptRoot=/home/site/wwwroot
COPY . /home/site/wwwroot
```

> [!IMPORTANT]
> The platform provides a CUDA runtime by default. If your workload requires specific CUDA versions or additional GPU libraries, include them in your container image. Verify compatibility with the [GPU software stack versions](gpu-serverless-overview.md#gpu-software-stack) supported by Azure Container Apps.

### Step 4: Create a Container Apps environment with GPU

```bash
# Create a resource group
az group create --name <RESOURCE_GROUP> --location swedencentral

# Create a Container Apps environment
az containerapp env create \
  --name <ENVIRONMENT_NAME> \
  --resource-group <RESOURCE_GROUP> \
  --location swedencentral

# Add a GPU workload profile
az containerapp env workload-profile add \
  --name <ENVIRONMENT_NAME> \
  --resource-group <RESOURCE_GROUP> \
  --workload-profile-name gpu-t4 \
  --workload-profile-type Consumption-GPU-NC8as-T4
```

### Step 5: Build and push the container image

Build your container image and push it to Azure Container Registry:

```bash
# Create a container registry (Premium SKU enables artifact streaming for faster image pulls)
az acr create --name <REGISTRY_NAME> --resource-group <RESOURCE_GROUP> --sku Premium

# Build and push the image
az acr build --registry <REGISTRY_NAME> --image my-gpu-function:v1 .
```

### Step 6: Deploy the function app with GPU

```bash
az containerapp create \
  --name <APP_NAME> \
  --resource-group <RESOURCE_GROUP> \
  --environment <ENVIRONMENT_NAME> \
  --image <REGISTRY_NAME>.azurecr.io/my-gpu-function:v1 \
  --registry-server <REGISTRY_NAME>.azurecr.io \
  --workload-profile-name gpu-t4 \
  --cpu 8.0 \
  --memory 56.0Gi \
  --ingress external \
  --target-port 80 \
  --kind functionapp \
  --min-replicas 0 \
  --max-replicas 5
```

The `--kind functionapp` flag enables Azure Functions integration. Setting `--min-replicas 0` enables scale-to-zero behavior for cost savings.

## Optimize cold start

GPU workloads often involve large container images and model files that increase cold start times. Use these strategies to reduce startup latency:

- **Artifact streaming**: Enable [artifact streaming](/azure/container-registry/container-registry-artifact-streaming) on your Azure Container Registry (Premium SKU required) to speed up image pulls.
- **Storage mounts**: Store large model files in an Azure storage mount instead of bundling them in the container image. For more information, see [Manage large downloads](cold-start.md#manage-large-downloads).
- **Minimum replicas**: Set `--min-replicas 1` to keep a warm instance, eliminating cold starts at the cost of continuous billing.

## Considerations

Keep the following limitations in mind:

- **One GPU per container**: Only one container in a function app can access the GPU. If you use sidecars, only the first container gets GPU access.
- **Workload profiles environment**: Serverless GPUs require a workload profiles environment. Consumption-only environments aren't supported.
- **Region availability**: GPU workload profiles are available in select regions. See [supported regions](gpu-serverless-overview.md#supported-regions).
- **Quota**: You must have GPU quota approved for your subscription. See [Request GPU quota](gpu-serverless-overview.md#request-serverless-gpu-quota).

## Monitor GPU usage

You can monitor GPU utilization and application performance through Azure Container Apps observability tools:

### Check GPU status

1. In the Azure portal, go to your container app.
1. Under **Monitoring**, select **Console**.
1. Select your replica and container, then choose **/bin/bash**.
1. Run `nvidia-smi` to view GPU memory usage, utilization, and running processes.

### View logs and metrics

1. Under **Monitoring**, select **Log stream** to view real-time container logs.
1. Under **Monitoring**, select **Metrics** to track CPU, memory, and replica count.
1. Use **Logs** to run KQL queries against your container app's log data.

For more information, see [Monitor Azure Container Apps](observability.md). You can also configure [Application Insights](../azure-functions/configure-monitoring.md) for deeper function execution telemetry.

## Next steps

- [Tutorial: Deploy AI image generation with serverless GPUs with Azure Functions on ACA](tutorial-gpu-image-generation.md)
- [Azure Functions on Azure Container Apps overview](functions-overview.md)
- [Using serverless GPUs in Azure Container Apps](gpu-serverless-overview.md)
- [Improve cold start for serverless GPUs](gpu-serverless-overview.md#improve-gpu-cold-start)
- [Monitor Azure Container Apps](observability.md)
