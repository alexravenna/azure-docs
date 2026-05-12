---
title: Use GPUs with Azure Functions on Azure Container Apps
description: Learn how to use serverless GPUs with Azure Functions on Azure Container Apps for compute-intensive AI and processing workloads.
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

When you host Azure Functions on [Azure Container Apps](../container-apps/overview.md), you can use [serverless GPUs](../container-apps/gpu-serverless-overview.md) to access NVIDIA A100 and T4 GPU resources on demand. Serverless GPUs scale to zero when idle, so you only pay for GPU compute when your functions are actively processing requests.

GPU support for Functions on Container Apps is available through Consumption GPU workload profiles in a workload profiles environment. You package your function code and GPU dependencies (such as CUDA libraries, AI models, or inference frameworks) into a custom container image, then deploy it to a Container Apps environment with a GPU workload profile.

## Key benefits

| Benefit | Description |
|---|---|
| **Event-driven GPU processing** | Combine Functions triggers and bindings with GPU compute for AI inference, image processing, or video transcription triggered by events from queues, HTTP, timers, and more. |
| **Scale to zero** | Serverless GPUs scale in to zero when idle, eliminating costs during inactive periods. |
| **Per-second billing** | Pay only for actual GPU compute time used during function execution. |
| **Automatic scaling** | Scale out to meet demand with KEDA-based event-driven autoscaling. |
| **Custom containers** | Package any GPU framework (PyTorch, TensorFlow, ONNX Runtime) alongside your function code. |
| **Full data governance** | Your data never leaves the container boundary. |

## Supported GPU types

Azure Functions on Container Apps supports the following GPU types through serverless GPU workload profiles:

| GPU type | Use case | vCPUs | Memory |
|---|---|---|---|
| **NVIDIA T4** | Cost-effective inference, image generation, smaller models | 8 | 56 GiB |
| **NVIDIA A100** | High-performance inference, large language models, training | 24 | 220 GiB |

For supported regions, see [Serverless GPU supported regions](../container-apps/gpu-serverless-overview.md#supported-regions).

## Prerequisites

Before you begin, make sure you have the following requirements:

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account).
- [GPU quota approved](../container-apps/gpu-serverless-overview.md#request-serverless-gpu-quota) for your subscription. Customers with enterprise agreements and pay-as-you-go subscriptions have A100 and T4 quota enabled by default.
- [Azure CLI](/cli/azure/install-azure-cli) version 2.62.0 or later.
- [Azure Functions Core Tools](functions-run-local.md) version 4.x.
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) for building container images.

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
> Your container image must include the necessary CUDA libraries and GPU frameworks for your workload. Verify compatibility with the [GPU software stack versions](../container-apps/gpu-serverless-overview.md#gpu-software-stack) supported by Azure Container Apps.

### Step 4: Build and push the container image

Build your container image and push it to Azure Container Registry:

```bash
# Create a container registry
az acr create --name <REGISTRY_NAME> --resource-group <RESOURCE_GROUP> --sku Basic

# Build and push the image
az acr build --registry <REGISTRY_NAME> --image my-gpu-function:v1 .
```

### Step 5: Create a Container Apps environment with GPU

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

## Common scenarios

### AI inference

Use GPU-accelerated Functions for real-time AI inference triggered by HTTP requests, queue messages, or events:

- **Image classification and object detection** — Process images uploaded to Azure Blob Storage using GPU-accelerated models.
- **Natural language processing** — Run LLMs or text analysis models triggered by Service Bus messages.
- **Speech and audio processing** — Transcribe audio files using Whisper or similar models.

### Batch processing

Process large volumes of data using GPU compute triggered by timers or queue events:

- **Video transcoding** — Transcode video files uploaded to storage.
- **Scientific computing** — Run simulations triggered on a schedule.
- **Data analytics** — Accelerate large-scale data transformations.

### Image generation

Generate images from text prompts using diffusion models. For a complete walkthrough, see [Tutorial: Deploy AI image generation with serverless GPUs with Azure Functions on ACA](../container-apps/tutorial-gpu-image-generation.md).

## Optimize cold start

GPU workloads often involve large container images and model files that increase cold start times. Use these strategies to reduce startup latency:

- **Artifact streaming**: Enable [artifact streaming](/azure/container-registry/container-registry-artifact-streaming) on your Azure Container Registry (Premium SKU required) to speed up image pulls.
- **Storage mounts**: Store large model files in an Azure storage mount instead of bundling them in the container image. For more information, see [Manage large downloads](../container-apps/cold-start.md#manage-large-downloads).
- **Minimum replicas**: Set `--min-replicas 1` to keep a warm instance, eliminating cold starts at the cost of continuous billing.

## Considerations

Keep the following limitations in mind:

- **One GPU per container**: Only one container in a function app can access the GPU. If you use sidecars, only the first container gets GPU access.
- **Container requirement**: GPU functions must be deployed as [custom containers](container-concepts.md). Code-only deployments aren't supported for GPU workloads.
- **Workload profiles environment**: Serverless GPUs require a workload profiles environment. Consumption-only environments aren't supported.
- **Region availability**: GPU workload profiles are available in select regions. See [supported regions](../container-apps/gpu-serverless-overview.md#supported-regions).
- **Quota**: You must have GPU quota approved for your subscription. See [Request GPU quota](../container-apps/gpu-serverless-overview.md#request-serverless-gpu-quota).

## Monitor GPU usage

You can monitor GPU utilization through the container console:

1. In the Azure portal, go to your container app.
1. Under **Monitoring**, select **Console**.
1. Run `nvidia-smi` to view GPU memory usage, utilization, and running processes.

For application-level monitoring, configure [Application Insights](monitor-functions.md) for your function app.

## Next steps

- [Tutorial: Deploy AI image generation with serverless GPUs with Azure Functions on ACA](../container-apps/tutorial-gpu-image-generation.md)
- [Azure Container Apps hosting of Azure Functions](functions-container-apps-hosting.md)
- [Using serverless GPUs in Azure Container Apps](../container-apps/gpu-serverless-overview.md)
- [Linux container support in Azure Functions](container-concepts.md)
