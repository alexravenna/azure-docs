---
title: 'Tutorial: Use Service Connector to connect a Spring Boot app to Kafka on Confluent Cloud
description: Create a Spring Boot app in Azure Spring Apps and connect it to Apache Kafka on Confluent Cloud by using Service Connector.
ms.devlang: java
author: maud-lv
ms.author: malev
ms.service: service-connector
ms.topic: tutorial
ms.date: 04/21/2026
ms.custom:
  - devx-track-extended-java
  - devx-track-azurecli
  - sfi-image-nochange
#customer intent: As a Java app developer and Confluent Cloud user, I want to learn how to use Service Connector to connect Azure Spring Apps and other services to my Confluent Cloud instance, so I can easily use Confluent Cloud in my apps.
---

# Tutorial: Connect a Spring Boot app to Kafka on Confluent Cloud

In this tutorial, you learn how to connect a Spring Boot application running on Azure Spring Apps to Apache Kafka on Confluent Cloud by using Service Connector. You complete the following tasks:

> [!div class="checklist"]
> * Create an Apache Kafka on Confluent Cloud instance.
> * Create a Spring Cloud application.
> * Build and deploy a Spring Boot app.
> * Connect Apache Kafka on Confluent Cloud to Azure Spring Apps using Service Connector.

>[!IMPORTANT]
>On March 17, 2025, all Azure Spring Apps Basic, Standard, and Enterprise plans entered a three-year retirement period, and will be fully retired on March 31, 2028. As of March 17, 2025, no new Azure Spring Apps plans or apps can be created. For more information, see the [Azure Spring Apps retirement announcement](/azure/spring-apps/basic-standard/retirement-announcement).

## Prerequisites

- An Azure subscription with write permissions for the tutorial resources, in an Azure region that [supports Service Connector](concept-region-support.md) and has sufficient [App Service support and quota](/azure/azure-resource-manager/management/azure-subscription-service-limits#azure-app-service-limits) for the tutorial resources.
- [Git](https://git-scm.com/) to clone the sample repo.

- Java 8 or a more recent version with long-term support.

- [Azure Cloud Shell](/azure/cloud-shell/get-started/classic) to run the tutorial steps, or if you prefer to run locally:
  1. Install [Azure CLI](/cli/azure/install-azure-cli) 2.30.0 or higher. To check your version, run `az --version`. To upgrade, run `az upgrade`.
  1. Sign in to Azure by using `az login` and following the prompts. If you have more than one subscription connected to your sign-in credentials, run `az account set --subscription <subscription-ID>` to select a subscription.

- The Service Connector `Microsoft.ServiceLinker` resource provider registered for your subscription. To register the provider, go to **Settings** > **Resource providers** in the Azure portal, or run the Azure CLI command `az provider register -n Microsoft.ServiceLinker`.

## Set up the sample repo

1. Run the following commands to clone the sample repo and change directories into the sample app project folder.

   ```bash
   git clone https://github.com/Azure-Samples/serviceconnector-springcloud-confluent-springboot/
   cd serviceconnector-springcloud-confluent-springboot
   ```

## Create a Kafka for Confluent Cloud organization

1. Create an instance of Apache Kafka for Confluent Cloud by following the instructions at [Quickstart: Create a Confluent resource in the Azure portal](/azure/partner-solutions/apache-kafka-confluent-cloud/create).

>[!IMPORTANT]
>To create a Confluent resource, you must [subscribe to Confluent Cloud](/azure/partner-solutions/apache-kafka-confluent-cloud/overview#subscribe-to-confluent-cloud).

1. Sign in to Confluent Cloud using the single sign-on (SSO) URL that Azure provides.

   :::image type="content" source="media/tutorial-java-spring-confluent-kafka/azure-confluent-sso-login.png" alt-text="Screenshot showing the link of Confluent Cloud SSO login using Azure portal." lightbox="media/tutorial-java-spring-confluent-kafka/azure-confluent-sso-login.png":::

1. In the default environment or a new one, create a Kafka cluster using the following settings:

   * Cluster type: Standard
   * Region/zones: The region that contains your Confluent Cloud resource, Single Zone
   * Cluster name: `cluster_1`.

   :::image type="content" source="media/tutorial-java-spring-confluent-kafka/confluent-cloud-env.png" alt-text="Screenshot showing the cloud environment of Apache Kafka on Confluent Cloud." lightbox="media/tutorial-java-spring-confluent-kafka/confluent-cloud-env.png":::

1. In **Cluster overview** > **Cluster settings**, make a note of the Kafka **Bootstrap server** URL.

   :::image type="content" source="media/tutorial-java-spring-confluent-kafka/confluent-cluster-setting.png" alt-text="Screenshot showing Cluster settings of Apache Kafka on Confluent Cloud." lightbox="media/tutorial-java-spring-confluent-kafka/confluent-cluster-setting.png":::

1. Under **Data integration** > **API Keys**, create API keys for the cluster using ** Add Key** with **Global access**. Make a note of the key and secret.

1. In **Topics** > **Add topic**, create a topic named `test` with partitions 6.

1. Under **default environment**, select the **Schema Registry** tab. Enable the Schema Registry and make a note of the **API endpoint**.

1. Create API keys for schema registry. Make a note of the key and secret.

## Create an Azure Spring Apps instance

>[!IMPORTANT]
>As of March 17, 2025, no new Azure Spring Apps plans or apps can be created. For more information, see [Azure Spring Apps retirement announcement](/azure/spring-apps/basic-standard/retirement-announcement).

1. Create an instance of Azure Spring Apps by following the instructions at [Quickstart: Deploy your first application to Azure Spring Apps](/azure/spring-apps/basic-standard/quickstart). Make sure to create the instance in a region that has [Service Connector support](concept-region-support.md).

1. Build the project using [Gradle](https://gradle.org/).

   ```Bash
   ./gradlew build
   ```

1. Create the app with a public endpoint assigned.

   ```azurecli
   az spring app create -n hellospring -s <service-instance-name> -g <your-resource-group-name> --assign-endpoint true
   ```

## Create the service connection

Use Azure CLI or the Azure portal to connect your Apache Kafka on Confluent Cloud instance to your Spring Cloud app.

> [!IMPORTANT]
> The connection authentication flow in this procedure requires a high degree of trust in the application, and carries risks not present in other flows. You should use this flow only when more secure flows, such as managed identities, aren't viable.

#### [CLI](#tab/Azure-CLI)

Run the following command, replacing the placeholders with your information.

* `<resource-group-name>`: Azure Spring Apps instance resource group.
* `<kafka-bootstrap-server-url>`: Kafka bootstrap server URL, for example `pkc-xxxx.eastus.azure.confluent.cloud:9092`.
* `<cluster-api-key>` and `<cluster-api-secret>`: Cluster API key and secret.
* `<kafka-schema-registry-endpoint>`: Kafka Schema Registry endpoint, for example `https://psrc-xxxx.westus2.azure.confluent.cloud`.
* `<registry-api-key>` and `<registry-api-secret>`: Kafka Schema Registry API key and secret.

```azurecli
az spring connection create confluent-cloud -g <spring-cloud-resource-group> --service <spring-cloud-service> --app <spring-cloud-app> --deployment <spring-cloud-deployment> --bootstrap-server <kafka-bootstrap-server-url> --kafka-key <cluster-api-key> --kafka-secret <cluster-api-secret> --schema-registry <kafka-schema-registry-endpoint> --schema-key <registry-api-key> --schema-secret <registry-api-secret>
```

#### [Portal](#tab/Azure-portal)

1. On your Spring App portal page, select **Service Connector** from the left navigation menu and enter the following settings.

   - **Service Type**: Select **Apache Kafka on Confluent cloud**.
   - **Name**: Generated unique connection name.
   - **Kafka bootstrap server URL**: Enter the bootstrap server URL, for example `pkc-xxxx.eastus.azure.confluent.cloud:9092`. 
   - **Cluster API Key**: Enter your cluster API key.
   - **Cluster API Secret**: Enter your cluster API secret.
   - **Create connection for schema registry**: Select to also create a connection to the schema registry. 
   - **Schema Registry endpoint**: Enter your Kafka Schema Registry endpoint.
   - **Schema Registry API Key**: Enter your Kafka Schema Registry API Key.
   - **Schema Registry API Secret**: Enter your Kafka Schema Registry API Secret.

1. Select **Review + Create** to review the connection settings, and then select **Create**.

---

## Deploy the JAR file

Run the following command to upload the JAR file *build/libs/java-springboot-0.0.1-SNAPSHOT.jar* to your Spring Cloud app.

```azurecli
az spring app deploy -n hellospring -s <service-instance-name> -g <resource-group-name>  --artifact-path build/libs/java-springboot-0.0.1-SNAPSHOT.jar
```

## Validate the Kafka data ingestion

1. On the Azure portal page for your Spring Cloud app, select **Browse** or **Default domain** near the top of the page.You should see the message **10 messages were produced to topic test**.

1. Go to the topic page in the Confluent portal to see production throughput.

   :::image type="content" source="media/tutorial-java-spring-confluent-kafka/confluent-sample-metrics.png" alt-text="Screenshot showing sample metrics." lightbox="media/tutorial-java-spring-confluent-kafka/confluent-sample-metrics.png":::

## Related content

- [Service Connector concepts](concept-service-connector-internals.md)
