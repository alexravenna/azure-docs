---
title: Enable features on a schedule in a Spring Boot application
titleSuffix: Azure App Configuration
description: Learn how to enable feature flags on a schedule in a Spring Boot application by using time window filters.
ms.service: azure-app-configuration
ms.devlang: java
author: mrm9084
ms.author: mametcal
ms.topic: how-to
ms.custom: mode-other, devx-track-java
ms.date: 03/23/2026
---

# Enable features on a schedule in a Spring Boot application

In this guide, you use the time window filter to enable a feature on a schedule for a Spring Boot application.

## Prerequisites

- An Azure account with an active subscription. [Create one for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An App Configuration store. [Create an App Configuration store](./quickstart-azure-app-configuration-create.md#create-an-app-configuration-store).
- A supported [Java Development Kit SDK](/java/azure/jdk) with version 17.
- [Apache Maven](https://maven.apache.org/download.cgi) version 3.0 or above.

## Add a feature flag

Add a feature flag called *Beta* to the App Configuration store and leave **Label** and **Description** with their default values. [Add a time window filter](./howto-timewindow-filter.md) to the *Beta* feature flag. For more information about how to add feature flags to a store using the Azure portal or the CLI, go to [Create a feature flag](./manage-feature-flags.md#create-a-feature-flag).

## Create a Spring Boot app

To create a new Spring Boot project:

1. Browse to the [Spring Initializr](https://start.spring.io).

1. Specify the following options:

   * Generate a **Maven** project with **Java**.
   * Specify a **Spring Boot** version that's equal to or greater than 3.0.
   * Specify the **Group** and **Artifact** names for your application. This article uses `com.example` and `demo`.

1. After you specify the previous options, select **Generate Project**. Download and extract the project to your local computer.

## Add feature management

1. Locate *pom.xml* in the root directory of your app and open it in a text editor.

1. Add the following to the list of `<dependencies>`:

    ```xml
    <dependency>
        <groupId>com.azure.spring</groupId>
        <artifactId>spring-cloud-azure-appconfiguration-config</artifactId>
    </dependency>
    <dependency>
        <groupId>com.azure.spring</groupId>
        <artifactId>spring-cloud-azure-feature-management</artifactId>
    </dependency>
    ```

1. Add the following `<dependencyManagement>` section to manage the Spring Cloud Azure library versions:

    ```xml
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.azure.spring</groupId>
                <artifactId>spring-cloud-azure-dependencies</artifactId>
                <version>7.1.0</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    ```

## Connect to an App Configuration store

1. Navigate to the `resources` directory of your app and open the `application.properties` or `application.yaml` file.

    You use the `DefaultAzureCredential` to authenticate to your App Configuration store. Follow the [instructions](./concept-enable-rbac.md#authentication-with-token-credentials) to assign your credential the **App Configuration Data Reader** role. Be sure to allow sufficient time for the permission to propagate before running your application.

    ### [Properties](#tab/properties)

    ```properties
    spring.config.import=azureAppConfiguration
    spring.cloud.azure.appconfiguration.stores[0].endpoint= ${AZURE_APPCONFIG_ENDPOINT}
    spring.cloud.azure.appconfiguration.stores[0].feature-flags.enabled=true
    ```

    ### [YAML](#tab/yaml)

    ```yaml
    spring:
      config:
        import: azureAppConfiguration
      cloud:
        azure:
          appconfiguration:
            stores:
              - endpoint: ${AZURE_APPCONFIG_ENDPOINT}
                feature-flags:
                  enabled: true
    ```

    ---

## Use the time window filter

The `spring-cloud-azure-feature-management` library includes the built-in `TimeWindowFilter`. This filter is registered automatically when you include the feature management dependency. You don't need to register it manually.

Update the `DemoApplication.java` file in the package directory of your app with the following code:

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

import com.azure.spring.cloud.feature.management.FeatureManager;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

    @Bean
    public CommandLineRunner runner(FeatureManager featureManager) {
        return args -> {
            for (int i = 0; i < 10; i++) {
                System.out.println("Beta is enabled: " + featureManager.isEnabled("Beta"));
                Thread.sleep(5000);
            }
        };
    }
}
```

## Time window filter in action

When you run the application, the configuration provider loads the *Beta* feature flag from Azure App Configuration. The result of the `isEnabled("Beta")` method will be printed to the console. If your current time is earlier than the start time set for the time window filter, the *Beta* feature flag will be disabled by the time window filter.

You'll see the following console outputs.

```bash
Beta is enabled: false
Beta is enabled: false
Beta is enabled: false
Beta is enabled: false
Beta is enabled: false
Beta is enabled: false
```

Once the start time has passed, you'll notice that the *Beta* feature flag is enabled by the time window filter.

You'll see the console outputs change as the *Beta* is enabled.

```bash
Beta is enabled: false
Beta is enabled: false
Beta is enabled: false
Beta is enabled: false
Beta is enabled: false
Beta is enabled: false
Beta is enabled: true
Beta is enabled: true
Beta is enabled: true
Beta is enabled: true
```

If recurrence is enabled when you set up the time window filter, the console outputs will change to `Beta is enabled: false` once your current time passes the end time you set in the time window filter. However, it will change to `Beta is enabled: true` again according to your recurrence settings and continue this pattern until the recurrence expiration time, if set.

## Next steps

To learn more about the feature filters, continue to the following documents.

> [!div class="nextstepaction"]
> [Enable conditional features with feature filters](./howto-feature-filters.md)

> [!div class="nextstepaction"]
> [Roll out features to targeted audience](./howto-targetingfilter.md)
