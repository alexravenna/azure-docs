---
title: Azure Service Bus JMS 2.0 developer guide
description: How to use the Java Message Service (JMS) 2.0 API to communicate with Azure Service Bus
ms.topic: article
ms.date: 06/16/2025
ms.custom:
  - devx-track-extended-java
  - sfi-ropc-nochange
---

# Azure Service Bus JMS 2.0 developer guide

This guide provides detailed information to help you succeed in communicating with Azure Service Bus by using the Java Message Service (JMS) 2.0 API.

As a Java developer, if you're new to Azure Service Bus, consider reading the following articles.

| Getting started | Concepts |
|----------------|-------|
| <ul> <li> [What is Azure Service Bus](service-bus-messaging-overview.md) </li> <li> [Queues, Topics, and Subscriptions](service-bus-queues-topics-subscriptions.md) </li> </ul> | <ul> <li> [Azure Service Bus - Premium tier](service-bus-premium-messaging.md) </li> </ul> |

## Java Message Service (JMS) programming model

The Java Message Service API programming model is shown in the following sections:

> [!NOTE]
>
>**Azure Service Bus Premium tier** supports JMS 1.1 and JMS 2.0.
> <br> <br>
> **Azure Service Bus - Standard** tier supports limited JMS 1.1 functionality. For more details, see [this documentation](service-bus-java-how-to-use-jms-api-amqp.md).
>

# [JMS 2.0 Programming model](#tab/JMS-20)

:::image type="content" source="./media/jms-developer-guide/java-message-service-20-programming-model.png" alt-text="Diagram showing JMS 2.0 Programming model." border="false":::

# [JMS 1.1 Programming model](#tab/JMS-11)

:::image type="content" source="./media/jms-developer-guide/java-message-service-11-programming-model.png" alt-text="Diagram showing JMS 1.1 Programming model." border="false":::

---

## JMS - Building blocks

Use the following building blocks to communicate with the JMS application.

> [!NOTE]
> This guide is adapted from the [Oracle Java EE 6 Tutorial for Java Message Service (JMS)](https://docs.oracle.com/javaee/6/tutorial/doc/bnceh.html).
>
> For a better understanding of the Java Message Service (JMS), refer to this tutorial.
>

### Connection factory

> [!NOTE]
> The `azure-servicebus-jms` library is available in two variants: `com.azure:azure-servicebus-jms` (version 2.0.0+) for **Jakarta EE** (`jakarta.jms.*`) and `com.microsoft.azure:azure-servicebus-jms` (version 1.0.x) for **Java EE** (`javax.jms.*`). For guidance on choosing the right artifact, see [Jakarta EE and javax support](how-to-use-java-message-service-20.md#jakarta-ee-and-javax-support).

The client uses the connection factory object to connect to the JMS provider. The administrator defines the connection factory with a set of connection configuration parameters.

Each connection factory is an instance of the `ConnectionFactory`, `QueueConnectionFactory`, or `TopicConnectionFactory` interface.

To simplify connecting to Azure Service Bus, the `ServiceBusJmsConnectionFactory`, `ServiceBusJmsQueueConnectionFactory`, and `ServiceBusJmsTopicConnectionFactory` classes implement these interfaces.

> [!IMPORTANT]
> Java applications that use the JMS 2.0 API can connect to Azure Service Bus by using the connection string or by using a `TokenCredential` to leverage Microsoft Entra backed authentication. When using Microsoft Entra backed authentication, ensure to [assign roles and permissions](service-bus-managed-service-identity.md#assign-a-service-bus-role-to-the-managed-identity) to the identity as needed.

# [System Assigned Managed Identity](#tab/system-assigned-managed-identity-backed-authentication)

Create a [system assigned managed identity](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md) on Azure, and use this identity to create a `TokenCredential`.

```java
TokenCredential tokenCredential = new DefaultAzureCredentialBuilder().build();
```

You can instantiate the connection factory with the following parameters:
   * Token credential - Represents a credential capable of providing an OAuth token.
   * Host - the hostname of the Azure Service Bus Premium tier namespace.
   * ServiceBusJmsConnectionFactorySettings property bag, which contains:
      * `connectionIdleTimeoutMS` - idle connection timeout in milliseconds.
      * `traceFrames` - boolean flag to collect AMQP trace frames for debugging.
      * Other configuration parameters.

Create the factory as shown in the following code. The token credential and host are required parameters, but the other properties are optional.

```java
String host = "<YourNamespaceName>.servicebus.windows.net";
ConnectionFactory factory = new ServiceBusJmsConnectionFactory(tokenCredential, host, null); 
```

# [User Assigned Managed Identity](#tab/user-assigned-managed-identity-backed-authentication)

Create a [user assigned managed identity](../active-directory/managed-identities-azure-resources/how-manage-user-assigned-managed-identities.md?pivots=identity-mi-methods-azp#create-a-user-assigned-managed-identity) on Azure, and use this identity to create a `TokenCredential`.

```java
TokenCredential tokenCredential = new DefaultAzureCredentialBuilder()
                                                .managedIdentityClientId("<clientIDOfUserAssignedIdentity>")
                                                .build();
```

You can instantiate the connection factory with the following parameters:
   * Token credential - Represents a credential capable of providing an OAuth token.
   * Host - the hostname of the Azure Service Bus Premium tier namespace.
   * ServiceBusJmsConnectionFactorySettings property bag, which contains:
      * `connectionIdleTimeoutMS` - idle connection timeout in milliseconds.
      * `traceFrames` - boolean flag to collect AMQP trace frames for debugging.
      * Other configuration parameters.

Create the factory as shown in the following code. The token credential and host are required parameters, but the other properties are optional.

```java
String host = "<YourNamespaceName>.servicebus.windows.net";
ConnectionFactory factory = new ServiceBusJmsConnectionFactory(tokenCredential, host, null); 
```

# [Service Principal](#tab/service-principal-backed-authentication)

Create a [service principal](authenticate-application.md#register-your-application-with-a-microsoft-entra-tenant) on Azure, and use this identity to create a `TokenCredential`.

```java
TokenCredential tokenCredential = new ClientSecretCredentialBuilder()
                .tenantId("")
                .clientId("")
                .clientSecret("")
                .build();
```

You can instantiate the connection factory with the following parameters:
   * Token credential - Represents a credential capable of providing an OAuth token.
   * Host - the hostname of the Azure Service Bus Premium tier namespace.
   * ServiceBusJmsConnectionFactorySettings property bag, which contains:
      * `connectionIdleTimeoutMS` - idle connection timeout in milliseconds.
      * `traceFrames` - boolean flag to collect AMQP trace frames for debugging.
      * Other configuration parameters.

Create the factory as shown in the following code. The token credential and host are required parameters, but the other properties are optional.

```java
String host = "<YourNamespaceName>.servicebus.windows.net";
ConnectionFactory factory = new ServiceBusJmsConnectionFactory(tokenCredential, host, null); 
```

# [Connection string authentication](#tab/connection-string-authentication)

You can instantiate the connection factory with the following parameters:
   * Connection string - the connection string for the Azure Service Bus Premium tier namespace.
   * ServiceBusJmsConnectionFactorySettings property bag, which contains:
      * `connectionIdleTimeoutMS` - idle connection timeout in milliseconds.
      * `traceFrames` - boolean flag to collect AMQP trace frames for debugging.
      * Other configuration parameters.

Create the factory as shown in the following example. The connection string is a required parameter, but the other properties are optional.

```java
ConnectionFactory factory = new ServiceBusJmsConnectionFactory(SERVICE_BUS_CONNECTION_STRING, null);
```

---

### JMS destination

A destination is the object a client uses to specify the target of the messages it produces and the source of the messages it consumes.

Destinations map to entities in Azure Service Bus - queues (in point to point scenarios) and topics (in pub-sub scenarios).

### Connections

A connection encapsulates a virtual connection with a JMS provider. With Azure Service Bus, it represents a stateful connection between the application and Azure Service Bus over AMQP.

Create a connection from the connection factory as shown in the following example:

```java
Connection connection = factory.createConnection();
```

### Sessions

A session is a single-threaded context for producing and consuming messages. Use it to create messages, message producers, and consumers. It also provides a transactional context to group sends and receives into an atomic unit of work.

Create a session from the connection object as shown in the following example:

```java
Session session = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);
```

> [!NOTE]
> JMS API doesn't support receiving messages from service bus queues or topics with messaging sessions enabled. 

#### Session modes

Create a session with any of the following modes.

| Session modes | Behavior |
| ----------------- | -------- |
|**Session.AUTO_ACKNOWLEDGE** | The session automatically acknowledges a client's receipt of a message either when the session successfully returns from a call to receive or when the message listener the session calls to process the message successfully returns.|
|**Session.CLIENT_ACKNOWLEDGE** |The client acknowledges a consumed message by calling the message's acknowledge method.|
|**Session.DUPS_OK_ACKNOWLEDGE**|This acknowledgment mode instructs the session to lazily acknowledge the delivery of messages.| 
|**Session.SESSION_TRANSACTED**|Pass this value as the argument to the method createSession(int sessionMode) on the Connection object to specify that the session should use a local transaction.| 

If you don't specify the session mode, the default is **Session.AUTO_ACKNOWLEDGE**.

### JMSContext

> [!NOTE]
> JMSContext is defined as part of the JMS 2.0 specification.
>

JMSContext combines the functionality provided by the connection and session objects. You create it from the connection factory object.

```java
JMSContext context = connectionFactory.createContext();
```

#### JMSContext modes

Like the **Session** object, you can create the JMSContext with the same acknowledge modes as described in [Session modes](#session-modes).

```java
JMSContext context = connectionFactory.createContext(JMSContext.AUTO_ACKNOWLEDGE);
```

If you don't specify a mode, the default is **JMSContext.AUTO_ACKNOWLEDGE**.

### JMS message producers

A message producer is an object that you create by using a JMSContext or a Session. Use it to send messages to a destination.

You can create it as a stand-alone object, as shown in the following example: 

```java
JMSProducer producer = context.createProducer();
```

Or you can create it at runtime when you need to send a message.

```java
context.createProducer().send(destination, message);
```

### JMS message consumers

A message consumer is an object that a JMSContext or a Session creates. Use it to receive messages sent to a destination. Create it as shown in the following example:

```java
JMSConsumer consumer = context.createConsumer(dest);
```

#### Synchronous receives via receive() method

The message consumer provides a synchronous way to receive messages from the destination through the `receive()` method.

If you don't specify arguments or timeout, or if you specify a timeout of `0`, the consumer blocks indefinitely unless the message arrives, or the connection is broken (whichever is earlier).

```java
Message m = consumer.receive();
Message m = consumer.receive(0);
```

When you provide a non-zero positive argument, the consumer blocks until that timer expires.

```java
Message m = consumer.receive(1000); // time out after one second.
```

#### Asynchronous receives with JMS message listeners

A message listener is an object that you use for asynchronous handling of messages on a destination. It implements the `MessageListener` interface, which contains the `onMessage` method where the specific business logic must live.

You must instantiate a message listener object and register it against a specific message consumer by using the `setMessageListener` method.

```java
Listener myListener = new Listener();
consumer.setMessageListener(myListener);
```

### Consuming from topics

You create [JMS Message Consumers](#jms-message-consumers) against a [destination](#jms-destination), which can be a queue or a topic.

Consumers on queues are simply client-side objects that live in the context of the Session (and Connection) between the client application and Azure Service Bus.

Consumers on topics, however, have two parts - 
   * A **client-side object** that lives in the context of the Session (or JMSContext), and
   * A **subscription** that is an entity on Azure Service Bus.

The subscriptions are documented [here](java-message-service-20-entities.md#java-message-service-jms-subscriptions) and can be one of the following types:
   * Shared durable subscriptions
   * Shared non-durable subscriptions
   * Unshared durable subscriptions
   * Unshared non-durable subscriptions

### JMS Queue Browsers

The JMS API provides a `QueueBrowser` object that the application can use to browse the messages in the queue and display the header values for each message.

You can create a Queue Browser by using the JMSContext, as shown in the following example:

```java
QueueBrowser browser = context.createBrowser(queue);
```

> [!NOTE]
> JMS API doesn't provide an API to browse a topic.
>
> This limitation exists because the topic itself doesn't store the messages. As soon as the message is sent to the topic, it's forwarded to the appropriate subscriptions.
>

### JMS message selectors

Receiving applications can use message selectors to filter the messages they receive. By using message selectors, the receiving application offloads the work of filtering messages to the JMS provider (in this case, Azure Service Bus) instead of taking that responsibility itself.

You can use selectors when creating any of the following consumers: 
   * Shared durable subscription
   * Unshared durable subscription
   * Shared non-durable subscription
   * Unshared non-durable subscription
   * Queue consumer
   * Queue browser

> [!NOTE]
> Service Bus selectors don't support `LIKE` and `BETWEEN` SQL keywords.

### Scheduled messages (delivery delay)

JMS 2.0 supports scheduling a message for future delivery by using the `setDeliveryDelay` method on a `MessageProducer` or `JMSProducer`. When you set this property, Service Bus accepts the message but only makes it visible to consumers after the delay period elapses.

```java
MessageProducer producer = session.createProducer(queue);

// Schedule a message for delivery 30 seconds from now
producer.setDeliveryDelay(30000);
producer.send(session.createTextMessage("Scheduled message"));
```

For a complete working sample, see [QueueScheduledSend.java](https://github.com/Azure/azure-servicebus-jms-samples/blob/sample/scheduled-messages/src/main/java/com/microsoft/azure/samples/QueueScheduledSend.java) in the azure-servicebus-jms-samples repository.

## AMQP disposition and Service Bus operation mapping

Here's how an AMQP disposition translates to a Service Bus operation:

```Output
ACCEPTED = 1; -> Complete()
REJECTED = 2; -> DeadLetter()
RELEASED = 3; (just unlock the message in service bus, will then get redelivered)
MODIFIED_FAILED = 4; -> Abandon() which increases delivery count
MODIFIED_FAILED_UNDELIVERABLE = 5; -> Defer()
```

## Summary

This developer guide shows how Java client applications that use Java Message Service (JMS) can connect to Azure Service Bus.

## Next steps

For more information about Azure Service Bus and details about Java Message Service (JMS) entities, see the following articles:
* [Service Bus - Queues, Topics, and Subscriptions](service-bus-queues-topics-subscriptions.md)
* [Service Bus - Java Message Service entities](service-bus-queues-topics-subscriptions.md#java-message-service-jms-20-entities)
* [AMQP 1.0 support in Azure Service Bus](service-bus-amqp-overview.md)
* [Service Bus AMQP 1.0 Developer's Guide](service-bus-amqp-dotnet.md)
* [Get started with Service Bus queues](service-bus-dotnet-get-started-with-queues.md)
* [Java Message Service API(external Oracle doc)](https://docs.oracle.com/javaee/7/api/javax/jms/package-summary.html)
* [Learn how to migrate from ActiveMQ to Service Bus](migrate-jms-activemq-to-servicebus.md)
