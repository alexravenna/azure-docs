---
title: 'Tutorial: Use Service Connector to connect Azure Spring Apps to Azure Database for MySQL'
description: Create a Java Spring Boot application and connect it to Azure Database for MySQL using Service Connector.
author: maud-lv
ms.author: malev
ms.service: service-connector
ms.topic: tutorial
ms.date: 04/22/2026
ms.custom: devx-track-azurecli, devx-track-extended-java
ms.devlang: azurecli
#customer intent: As a Java Spring Cloud app developer and MySQL database user, I want to learn how to use Service Connector to connect Java Spring Boot apps to my Azure Database for MySQL databases, so I can easily use my database data in my apps.
---

# Tutorial: Connect a Java Spring Boot application to Azure Database for MySQL

This tutorial uses Azure CLI to complete the following tasks:

> [!div class="checklist"]
> * Create an Azure Database for MySQL server and database.
> * Create and provision an instance of Azure Spring Apps.
> * Build and deploy a Java app to Azure Spring Apps.
> * Use Service Connector to integrate Azure Spring Apps with Azure Database for MySQL.

>[!IMPORTANT]
>On March 17, 2025, Azure Spring Apps entered a three-year retirement period, and will be fully retired on March 31, 2028. As of March 17, 2005, no new Azure Spring Apps plans or apps can be created. For more information, see the [Azure Spring Apps retirement announcement](/azure/spring-apps/basic-standard/retirement-announcement).

Because of the [Azure Spring Apps retirement](/azure/spring-apps/basic-standard/retirement-announcement), you can run the Azure Spring Apps parts of this tutorial only if you have an preexisting Azure Spring Apps plan, and then only until March 31, 2028. For other ways to connect Java JBoss apps to Azure Database for MySQL using Service Connector, see [Tutorial: Connect to a MySQL database from Java JBoss EAP using passwordless connection](tutorial-java-jboss-connect-managed-identity-mysql-database.md) and [Integrate Azure Database for MySQL with Service Connector](how-to-integrate-mysql.md).

## Prerequisites

- An Azure subscription with write permissions for the tutorial resources, in an Azure region that [supports Service Connector](concept-region-support.md) and has sufficient [App Service support and quota](/azure/azure-resource-manager/management/azure-subscription-service-limits#azure-app-service-limits) for the tutorial resources.

- An existing Azure Spring Apps plan. Due to the upcoming [Azure Spring Apps retirement](/azure/spring-apps/basic-standard/retirement-announcement), you can run the Azure Spring Apps parts of this tutorial only if you have an preexisting Azure Spring Apps plan, and only until March 31, 2028.

- [Git](https://git-scm.com/) to clone the sample repo.

- Java 8 or a more recent version with long-term support.

- [Azure Cloud Shell](/azure/cloud-shell/get-started/classic) to run the tutorial steps, or if you prefer to run locally:
  1. Install [Azure CLI](/cli/azure/install-azure-cli) 2.0.67 or higher. To check your version, run `az --version`. To upgrade, run `az upgrade`.
  1. Sign in to Azure by using `az login` and following the prompts. If you have more than one subscription connected to your sign-in credentials, run `az account set --subscription <subscription-ID>` to select a subscription.

- The Service Connector `Microsoft.ServiceLinker` resource provider registered for your subscription. To register the provider, go to **Settings** > **Resource providers** in the Azure portal, or run the Azure CLI command `az provider register -n Microsoft.ServiceLinker`.

## Set up your environment

1. In Cloud Shell, define the following environment variables for the tutorial, replacing the `<region>` placeholder with a valid value. `LOCATION` must be an Azure region where your subscription has sufficient quota to create the Azure resources and no restrictions on any of the services.

   ```bash
   LOCATION="<region>"
   RESOURCE_GROUP="ServiceConnector-tutorial-mysqlf"
   ```

1. Create a [resource group](/azure/azure-resource-manager/management/overview#terminology) to contain all the project resources.

   ```azurecli
   az group create --name $RESOURCE_GROUP --location $LOCATION
   ```

## Create an Azure Database for MySQL

Create an Azure Database for MySQL server and database in your subscription. The server uses the following parameters and values:

- The MySQL host name must be unique across all of Azure.
- The default SKU is a General Purpose Gen5 server with two vCores. Standard_B1ms SKU is used by default. For more information about pricing, see [Azure Database for MySQL pricing](https://azure.microsoft.com/pricing/details/mysql/).
- The `admin-user` can't be `azure_superuser`, `azure_pg_admin`, `admin`, `administrator`, `root`, `guest`, or `public`, and can't start with `pg_`. 
- The `admin-password` must contain 8-128 Latin uppercase letters, Latin lowercase letters, numbers, or non-alphanumeric characters, excluding `username`.
- The following default values are set unless you manually override them:
  - `backup-retention`: 7 days
  - `geo-redundant-backup`: Disabled
  - `ssl-enforcement: Enabled
  - `version: MySQL major version 5.7

For more information, see [az mysql flexible-server create](/cli/azure/mysql/flexible-server#az-mysql-flexible-server-create).

>[!IMPORTANT]
>The connection authentication using secrets requires a high degree of trust in the application, and carries risks not present in other flows. You should use this flow only when more secure flows, such as managed identities, aren't viable.

Run the following command to create the Azure Database for MySQL server and database.

```azurecli
export MYSQL_ADMIN_USER=azureuser
export MYSQL_ADMIN_PASSWORD="AdminPassword1"
export RAND_ID=$RANDOM
export MYSQL_HOST="mysqlf-$RAND_ID"
az mysql flexible-server create \
    --name $MYSQL_HOST \
    --database-name mysqlf-db \
    --resource-group $RESOURCE_GROUP \
    --location $LOCATION \
    --admin-user $MYSQL_ADMIN_USER \
    --admin-password $MYSQL_ADMIN_PASSWORD \
    --public-access 0.0.0.0 \
    --tier Burstable \
    --sku-name Standard_B1ms \
    --storage-size 32
```

## Create and build the Azure Spring Apps instance and app

The following procedure uses the Azure CLI extension to create and provision an instance of Azure Spring Apps. Because of the [Azure Spring Apps retirement](/azure/spring-apps/basic-standard/retirement-announcement), you can run this procedure only if you have an preexisting Azure Spring Apps plan, and then only until March 31, 2028. 

1. Update Azure CLI with the Azure Spring Apps extension.

    ```azurecli
    az extension update --name spring
    ```

1. Create an instance of Azure Spring Apps. The name must be 4-32 lowercase letters, numbers, and hyphens. The first character must be a letter and the last character must be either a letter or a number.

    ```azurecli
    az spring create -n my-azure-spring -g $RESOURCE_GROUP
    ```

1. Create the app with public endpoint assigned.

    ```azurecli
    az spring app create -n hellospring -s my-azure-spring -g $RESOURCE_GROUP --assign-endpoint true
    ```

1. Run the `az spring connection create` command to connect the Azure Spring Apps application to the MySQL database. Replace the placeholders below with your own information.

    ```azurecli
    az spring connection create mysql-flexible \
        --resource-group $RESOURCE_GROUP \
        --service my-azure-spring \
        --app hellospring \
        --target-resource-group $RESOURCE_GROUP \
        --server $MYSQL_HOST \
        --database mysqlf-db \
        --secret name=$MYSQL_ADMIN_USER secret=$MYSQL_ADMIN_PASSWORD
    ```

1. Clone the sample repo.

    ```bash
    git clone https://github.com/Azure-Samples/serviceconnector-springcloud-mysql-springboot.git
    ```

1. Change directories into the project directory.

    ```bash
    cd serviceconnector-springcloud-mysql-springboot
    ```

1. Build the project using Maven.

    ```bash
    mvn clean package -DskipTests 
    ```

1. Deploy the *target/demo-0.0.1-SNAPSHOT.jar* JAR file for the app.

    ```azurecli
    az spring app deploy \
        --name hellospring \
        --service my-azure-spring \
        --resource-group $RESOURCE_GROUP \
        --artifact-path target/demo-0.0.1-SNAPSHOT.jar
    ```

1. Run the following command to query app status after deployment:

    ```azurecli
    az spring app list  --resource-group $RESOURCE_GROUP --service my-azure-spring --output table
    ```

    You should see the following output:

    ```output
    Name         Location    ResourceGroup                     Public Url                                                 Production Deployment    Provisioning State    CPU    Memory    Running Instance    Registered Instance    Persistent Storage    Bind Service Registry    Bind Application Configuration Service
    -----------  ----------  --------------------------------  ---------------------------------------------------------  -----------------------  --------------------  -----  --------  ------------------  ---------------------  --------------------  -----------------------  ----------------------------------------
    hellospring  eastus      ServiceConnector-tutorial-mysqlf  https://my-azure-spring-hellospring.azuremicroservices.io  default                  Succeeded             1      1Gi       1/1                 0/1                    -                     -

    ```

## Related content

- [Service Connector concepts](concept-service-connector-internals.md)
- [Tutorial: Connect to a MySQL database from Java JBoss EAP using passwordless connection](tutorial-java-jboss-connect-managed-identity-mysql-database.md)
- [Integrate Azure Database for MySQL with Service Connector](how-to-integrate-mysql.md)
