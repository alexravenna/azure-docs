---
title: Process and route data with data flows
description: Learn about data flows and how to process and route data in Azure IoT Operations.
author: sethmanheim
ms.author: sethm
ms.service: azure-iot-operations
ms.subservice: azure-data-flows
ms.topic: concept-article
ms.date: 05/08/2026

#CustomerIntent: As an operator, I want to understand how I can use data flows to connect data sources.
---

# Process and route data with data flows

Data flows simplify the setup of data paths to move, transform, and enrich data. By using data flows, you can connect various data sources and perform data operations. The data flow component is part of Azure IoT Operations, which you deploy as an Azure Arc extension. You configure a data flow by using the operations experience web UI, the Azure CLI, or Azure Resource Manager templates.

You can write configurations for various use cases, such as:

- Transform data and send it back to MQTT.
- Transform data and send it to the cloud.
- Send data to the cloud or edge without transformation.

Data flows aren't limited to the region where you deploy the IoT Operations instance. You can use data flows to send data to cloud endpoints in different regions.

> [!NOTE]
> Data flows replace the preview-only **Data Processor** component from early Azure IoT Operations releases. The `--include-dp` parameter on `az iot ops init` was removed and is no longer required—the data flows components deploy automatically.

## Key features

This section describes the key features of data flows.

### Data processing and routing

Data flows enable the ingestion, processing, and routing of the messages to specified sinks. You can specify:

- **Sources**: Where you ingest messages from.
- **Destinations**: Where you drain messages to, including support for dynamic topic routing based on message content for MQTT endpoints.
- **Transformations (optional)**: Configuration for data processing operations.

### Transformation capabilities

You can apply transformations to data during the processing stage to perform various operations. These operations can include:

- **Computing new properties**: Based on existing properties in the message.
- **Renaming properties**: To standardize or clarify data.
- **Converting units**: Convert values to different units of measurement.
- **Standardizing values**: Scale property values to a user-defined range.
- **Contextualizing data**: Add reference data to messages for enrichment and driving insights.

> [!TIP]
> For richer processing capabilities including conditional routing, time-based aggregation, and composable transform pipelines, see [Data flow graphs](concept-dataflow-graphs.md).

### Configuration and deployment

Specify the configuration by using the operations experience web UI, the Azure CLI, or Azure Resource Manager templates. Based on this configuration, the data flow operator creates data flow instances to ensure high availability and reliability.

## Benefits

- **Simplified setup**: Easily connect data sources and destinations.
- **Flexible transformations**: Perform a wide range of data operations.
- **Scalable configuration**: Use Azure tools for scalable and manageable configurations.
- **High availability**: Kubernetes native resource ensures reliability.

By using data flows, you can efficiently manage your data paths. You can ensure that data is accurately sent, transformed, and enriched to meet your operational needs.

## Schema registry

Schema registry, a feature provided by Azure Device Registry, is a synchronized repository in the cloud and at the edge. The schema registry stores the definitions of messages coming from edge assets, and then exposes an API to access those schemas at the edge. Southbound connectors like the connector for OPC UA can create message schemas and add them to the schema registry, or you can upload schemas to the operations experience web UI.

Data flows use message schemas to transform the message into the format expected by the destination endpoint.

For more information, see [Understand message schemas](./concept-schema-registry.md).

## Resiliency when destination endpoints are unavailable

When you use the local MQTT broker as a source endpoint in a data flow, either directly or indirectly through assets that publish to it, the data flow receives messages from the MQTT broker as a subscriber. The data flow acknowledges each source message after the message is successfully processed and delivered to the destination, or after the message is intentionally dropped because of filtering, schema validation, or message expiry.

If the destination endpoint, such as Azure Event Hubs, Fabric Real-Time Intelligence, Kafka, or another cloud service, is unavailable, delivery can't complete. In this case, the data flow doesn't acknowledge the source message. The MQTT broker keeps the message in the subscriber queue and the data flow retries delivery. When connectivity is restored, the data flow sends queued messages to the destination and acknowledges them after successful delivery.

```text
Publisher or asset -> MQTT broker -> Data flow -> Destination endpoint

1. The MQTT broker delivers a message to the data flow.
2. The data flow sends the message to the destination endpoint.
3. If the send succeeds, the data flow acknowledges the source message and the broker removes it from the subscriber queue.
4. If the send fails, the data flow doesn't acknowledge the source message. The broker keeps it queued.
5. The data flow retries delivery until it succeeds, the message expires, or a configured limit applies.
```

> [!IMPORTANT]
> Data flow buffering is bounded. Queued messages are subject to the MQTT broker memory profile, subscriber queue limits, disk-backed message buffer size, persistence configuration, and message or session expiry. Configure these settings for the maximum outage duration and throughput you need to tolerate.

### Data protection configuration layers

Use the following configuration layers to control how messages are buffered and protected during destination outages.

| Layer | What it protects against | Default behavior | Customer action |
| --- | --- | --- | --- |
| Data flow retry and withheld acknowledgment | Temporary destination or network outage | Built in for local MQTT broker and asset-backed sources | No configuration required |
| MQTT broker subscriber queue | Messages received by a data flow subscription but not yet acknowledged | Stored in memory | Configure memory profile and subscriber queue limits |
| Disk-backed message buffer | Large temporary backlogs that exceed available memory | Disabled | Configure the broker at deployment with a disk-backed message buffer |
| MQTT broker persistence | Broker or pod restart while messages are queued | Disabled by default | Enable broker persistence and subscriber queue persistence |
| Data flow `requestDiskPersistence` | Per-data-flow request for persistent subscriber queue storage | Disabled | Enable `requestDiskPersistence` on the data flow or data flow graph, and enable dynamic subscriber queue persistence on the broker |
| Message and session expiry | Bounded storage and replay behavior | Configurable | Set expiry and limits based on your loss tolerance and outage window |

The local MQTT broker subscriber queue is stored in memory by default. You can configure the MQTT broker to use disk in two different ways:

- **Disk-backed message buffer**: Uses disk as a spillover buffer when queues grow beyond available memory. This setting helps with larger temporary backlogs, but it isn't the same as durable persistence across broker restarts.
- **MQTT broker persistence**: Persists selected broker data, including subscriber queues, to disk so queued messages can survive restarts or power loss.

For more information, see:

- [Configure broker settings for high availability, scaling, and memory usage](../manage-mqtt-broker/howto-configure-availability-scale.md)
- [Configure disk-backed message buffer behavior](../manage-mqtt-broker/howto-disk-backed-message-buffer.md)
- [Configure MQTT broker persistence](../manage-mqtt-broker/howto-broker-persistence.md)
- [Configure disk persistence for data flows](howto-configure-disk-persistence.md)
- [Configure broker MQTT client options](../manage-mqtt-broker/howto-broker-mqtt-client-options.md#subscriber-queue-limit)

### Choose a buffering configuration

Choose a configuration based on the outage duration and durability requirements for your workload:

- For short transient cloud or network outages, the default in-memory subscriber queue might be enough.
- For higher throughput or longer temporary outages, configure the disk-backed message buffer.
- For restart or power-loss protection, enable MQTT broker persistence and subscriber queue persistence, then enable `requestDiskPersistence` on the data flow or data flow graph.
- For bounded storage environments, configure subscriber queue limits, message expiry, and monitoring so the broker enforces queue limits and drops or rejects messages according to your policy.

### Example: Destination outage

Assume that you create a data flow by using the default local MQTT broker as the source endpoint and Azure Event Hubs as the destination endpoint. If connectivity between the data flow and Azure Event Hubs is lost, the data flow retries sends and doesn't acknowledge source messages. The MQTT broker queues the unacknowledged messages. With default settings, the queue is stored in memory. With disk-backed message buffer, the queue can spill to disk. With broker persistence and data flow `requestDiskPersistence`, queued messages can survive broker restarts, subject to the configured persistence, expiry, and storage limits.

## Related content

- [Data flows vs. data flow graphs](overview-dataflow-comparison.md)
- [Data flow graphs overview](concept-dataflow-graphs.md)
- [Create a data flow](howto-create-dataflow.md)
- [Configure a data flow source](howto-configure-dataflow-source.md)
- [Create a data flow endpoint](howto-configure-dataflow-endpoint.md)
- [Tutorial: Send messages from assets to the cloud using a data flow](../end-to-end-tutorials/tutorial-upload-messages-to-cloud.md)
