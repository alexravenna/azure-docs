---
title: Azure Container Apps Express overview
description: Learn about Azure Container Apps Express, a developer-first platform that lets you deploy containerized web apps to Azure with minimal configuration and rapid provisioning.
ms.topic: overview
ms.date: 05/01/2026
author: craigshoemaker
ms.author: cshoe
ms.service: container-apps
ms.custom: devx-track-container-apps
---

# Azure Container Apps Express overview

Azure Container Apps Express gives you the fastest way to deploy containerized web applications to Azure. With opinionated defaults and a minimal configuration surface, Express is a developer-first and agent-first platform designed to get your web apps running in the cloud as fast as possible. Rapid provisioning and scale-from-zero make Express an ideal host for AI-powered applications and agent backends.

## Key capabilities

Express removes infrastructure decisions so you can focus on building your application.

| Capability | Details |
|---|---|
| **High-speed launch** | Deploy in minutes with no infrastructure tuning required. Scaling behavior is built in from the start. |
| **Simple, powerful apps** | Run HTTP-first workloads including APIs, SaaS frontends, AI gateways, and event-driven web backends. |
| **Automatic elasticity** | Scale from zero to hyperscale automatically. Express is designed for unpredictable traffic patterns, and scaling is handled for you. |
| **Scale from zero** | Your app scales down to zero when idle and back up on demand, so you only pay for what you use. |
| **High-speed startup** | Optimized cold start ensures your app is ready to serve traffic quickly. |
| **Opinionated defaults** | Sensible defaults are applied automatically so you don't have to configure infrastructure settings. |
| **Minimal configuration surface** | Fewer decisions to make means faster time to production. |
| **Developer velocity** | Spend less time on infrastructure and more time writing code. |

## Common scenarios

Express is ideal for HTTP-based web workloads where speed of deployment and simplicity are priorities.

- **SaaS applications**: Launch SaaS products without worrying about scaling infrastructure.
- **AI app frontends**: Deploy AI-powered interfaces and gateways that scale with demand.
- **Developer tools**: Ship internal and external dev tools with zero-config deployment.
- **Web dashboards**: Build internal analytics, monitoring, and admin panels with instant availability.
- **Startups and new projects**: Go from idea to production in minutes. Prototype fast, and scale as you grow.
- **Rapid prototyping**: Build and validate ideas quickly, then keep running in production without replatforming.

## How Express works

Express simplifies the deployment experience by removing the need to create and manage a Container Apps environment. You deploy your app directly, and the platform provisions the underlying resources for you.

- **No environment to manage**: Deploy your container app without creating or configuring an environment. Express handles infrastructure allocation automatically.
- **Consumption-based compute**: Express runs on consumption CPU with pay-as-you-go pricing. Your apps scale to zero when idle, so you only pay for the compute your app uses.
- **Opinionated defaults**: Configuration decisions like scaling rules, networking, and resource allocation are handled by the platform with production-ready defaults.
- **Request-driven duration**: Compute runs when your app receives requests and scales down when traffic stops.
- **Optimized cold start**: Express automatically optimizes cold-start behavior, so your app is ready to serve traffic quickly after scaling from zero.

## When to use Express

Use the following table to determine if Express is the right fit for your workload.

| Scenario | Use Express | Alternative |
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

Keep these important points in mind when using Express:

- **HTTP workloads only**: Express supports web apps and APIs that communicate over HTTP. TCP-based workloads aren't supported.
- **Consumption CPU compute**: Express runs on consumption-based CPU compute. GPU workloads require standard Container Apps with [dedicated workload profiles](workload-profiles-overview.md).
- **Opinionated configuration**: Express uses opinionated defaults with a minimal configuration surface. If you need fine-grained control over compute, networking, or cold-start behavior, use standard Container Apps with a [workload profiles environment](environment.md).
- **Feature availability**: Express offers a focused set of features at launch. Some capabilities available in standard Container Apps environments, such as custom virtual networks, Dapr integration, and built-in service discovery, aren't available in Express.

## Supported features

Express launches with the following set of supported features. This list is updated as new capabilities are enabled.

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

The following features aren't available in Express at launch:

- Secrets
- Secrets from Key Vault
- Autoscale (KEDA-based)
- Managed identity (app runtime)
- Managed identity (image pull)
- Virtual network (VNet) integration
- Quota management
- Health probes
- Exec access
- Easy Auth
- Metrics (Azure Monitor)
- Custom domain (managed certificate)
- IP restrictions
- CORS
- Logs (Azure Monitor)
- Session affinity
- Sidecar containers
- Init containers
- Volume mounts
- Ephemeral storage
- GPU
- Insecure HTTP ingress
- Additional ports
- App-to-app communication
- Debug console
- Deployment labels
- Language stacks
- Multi revision/traffic splitting
- Resiliency policies
- Source-to-cloud deployment
- TCP protocol
- Aspire
- Maintenance windows
- OpenTelemetry
- Premium ingress
- Private endpoints
- Workload profiles
- Peer-to-peer encryption
- Jobs
- Single revision management
- Custom domain (bring your own certificate)
- Environment custom domain suffix
- Azure file storage
- Zone redundancy
- App-to-app internal FQDN
- Internal vs. external apps

## Region availability

During the initial preview, Express is available only in the **West Central US** region. Support for more regions will be added in future releases.

## Get started

To deploy your first Express container app, see [Quickstart: Deploy a container app with the Azure CLI](quickstart-container-apps.md).

For issues or feedback during the preview, file an issue on the [Azure Container Apps GitHub repository](https://github.com/microsoft/azure-container-apps/). Start the issue title with **[EPP]** to identify it as an Express Preview issue.

Example: `[EPP] Deployment fails when using custom environment variables`

## Next steps

> [!div class="nextstepaction"]
> [Quickstart: Deploy a container app with the Azure CLI](quickstart-container-apps.md)

- Learn about [Container Apps environments](environment.md)
- Explore [Container Apps jobs](jobs.md)
- Review [Container Apps billing](billing.md)
