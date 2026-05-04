---
title: Azure Container Apps Express Overview
description: Learn about Azure Container Apps express, a developer-first platform that lets you deploy containerized web apps to Azure with minimal configuration and rapid provisioning.
ms.topic: overview
ms.date: 05/04/2026
author: craigshoemaker
ms.author: cshoe
ms.service: azure-container-apps
---

# Azure Container Apps express overview

Azure Container Apps express provides the fastest way to deploy containerized web applications to Azure. With opinionated defaults and a minimal configuration surface, express is a developer-first and agent-first platform designed to get your web apps running in the cloud as fast as possible.

With express, you can directly create a container app without having to create a environment first. The rapid provisioning and scale-from-zero features make express an ideal host for AI-powered applications and agent backends.

:::image type="content" source="media/express-overview/azure-container-apps-express-welcome.png" alt-text="Screenshot of the Azure Container Apps express welcome screen.":::

## Key capabilities

The express deployment model removes infrastructure decisions so you can focus on building your application.

| Capability | Details |
|---|---|
| **High-speed launch** | Deploy in minutes with no infrastructure tuning required. Scaling behavior is built in from the start. |
| **Simple, powerful apps** | Run HTTP-first workloads including APIs, SaaS frontends, AI gateways, and event-driven web backends. |
| **Automatic elasticity** | Scale from zero to hyperscale automatically. The platform is designed for unpredictable traffic patterns, and scaling is handled for you. |
| **Scale from zero** | Your app scales down to zero when idle and back up on demand, so you only pay for what you use. |
| **High-speed startup** | Optimized cold start ensures your app is ready to serve traffic quickly. |
| **Opinionated defaults** | Sensible defaults are applied automatically so you don't have to configure infrastructure settings. |
| **Minimal configuration surface** | Fewer decisions to make means faster time to production. |
| **Developer velocity** | Spend less time on infrastructure and more time writing code. |

## Common scenarios

The express deployment model works best for HTTP-based web workloads where speed of deployment and simplicity matter most.

- **SaaS applications**: Launch SaaS products without worrying about scaling infrastructure.

- **AI app frontends**: Deploy AI-powered interfaces and gateways that scale with demand.

- **Developer tools**: Ship internal and external dev tools with zero-config deployment.

- **Web dashboards**: Build internal analytics, monitoring, and admin panels with instant availability.

- **Startups and new projects**: Go from idea to production in minutes. Prototype fast, and scale as you grow.

- **Rapid prototyping**: Build and validate ideas quickly, then keep running in production without replatforming.

## How express works

The express deployment model simplifies the deployment experience by removing the need to create and manage a Container Apps environment. You deploy your app directly, and the platform provisions the underlying resources for you.

- **No environment to manage**: Deploy your container app without creating or configuring an environment. The platform handles infrastructure allocation automatically. This environment-less model reduces setup steps and makes it easy to get started.

- **Consumption-based compute**: Express apps run on consumption CPU with pay-as-you-go pricing. Your apps scale to zero when idle, so you only pay for the compute your app uses.

- **Opinionated defaults**: The platform handles configuration decisions like scaling rules, networking, and resource allocation with production-ready defaults.

- **Request-driven duration**: Compute runs when your app receives requests and scales down when traffic stops.

- **Optimized cold start**: The platform automatically optimizes cold-start behavior, so your app is ready to serve traffic quickly after scaling from zero.

- **Dedicated management UI**: Express apps are managed [through a dedicated UI](https://containerapps.azure.com/) separate from the Azure portal. When you create or manage an express app, the platform directs you to this streamlined interface instead of the standard Azure portal experience.

    :::image type="content" source="media/express-overview/azure-container-apps-express-create.png" alt-text="Screenshot of Azure Container Apps express create experience.":::

## When to use express

Use the following table to determine if express is the right fit for your workload.

| Scenario | Use express | Alternative |
|---|---|---|
| Web apps and REST APIs | ✅ Yes | |
| SaaS frontends and AI gateways | ✅ Yes | |
| Rapid prototyping and startups | ✅ Yes | |
| Web dashboards and admin panels | ✅ Yes | |
| GPU workloads | ❌ No | Use [serverless GPUs](gpu-serverless-overview.md) with dedicated workload profiles |
| TCP-based services | ❌ No | Use standard [Container Apps environments](environment.md) |
| Jobs and batch processing | ❌ No | Use [Container Apps jobs](jobs.md) |
| Microservices with service discovery | ❌ No | Use standard [Container Apps environments](environment.md) |

## Considerations

Keep these important points in mind when using express:

- **HTTP workloads only**: Express supports web apps and APIs that communicate over HTTP. TCP-based workloads aren't supported.

- **Consumption CPU compute**: Express apps run on consumption-based CPU compute. GPU workloads require standard Container Apps with [dedicated workload profiles](workload-profiles-overview.md).

- **Opinionated configuration**: The express model uses opinionated defaults with a minimal configuration surface. If you need fine-grained control over compute, networking, or cold-start behavior, use standard Container Apps with a [workload profiles environment](environment.md).

- **Feature availability**: Express offers a focused set of features at launch. Some capabilities available in standard Container Apps environments, such as custom virtual networks, Dapr integration, and built-in service discovery, aren't available in express.

## Supported features

The express deployment model supports the following features. This list is updated as Microsoft enables new capabilities.

### Supported features

- Scale to zero
- Image deployment (anonymous and token-based)
- Multi replica
- Environment variables
- Enable ingress
- Portal experience
- Log streaming
- Region restriction
- Logs (Log Analytics)
- Rolling updates (partial support)

### Unsupported features

The following features aren't available in express at launch:

:::row:::
   :::column:::
   - Additional ports
   - App-to-app communication
   - App-to-app internal FQDN
   - Aspire
   - Autoscale (KEDA-based)
   - Azure file storage
   - CORS
   - Custom domain (bring your own certificate)
   - Custom domain (managed certificate)
   - Debug console
   - Deployment labels
   - Easy Auth
   - Ephemeral storage
   - Environment custom domain suffix
   - Exec access
   - GPU
   - Health probes
   - Init containers
   - Insecure HTTP ingress
   - Internal vs. external apps
   - IP restrictions
   - Jobs
   - Language stacks
   :::column-end:::
   :::column:::
   - Logs (Azure Monitor)
   - Maintenance windows
   - Managed identity (app runtime)
   - Managed identity (image pull)
   - Metrics (Azure Monitor)
   - Multi revision/traffic splitting
   - OpenTelemetry
   - Peer-to-peer encryption
   - Premium ingress
   - Private endpoints
   - Quota management
   - Resiliency policies
   - Secrets
   - Secrets from Key Vault
   - Session affinity
   - Sidecar containers
   - Single revision management
   - Source-to-cloud deployment
   - TCP protocol
   - Virtual network (VNet) integration
   - Volume mounts
   - Workload profiles
   - Zone redundancy
   :::column-end:::
:::row-end:::

## Region availability

During the initial preview, Express is available only in the **West Central US** region. Support for more regions will be added in future releases.
