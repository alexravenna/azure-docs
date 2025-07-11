---
title: Run automated tests by using Azurite
titleSuffix: Azure Storage
description: Learn how to write automated tests against private endpoints for Azure Blob Storage by using Azurite.
services: storage
author: ikivanc

ms.service: azure-blob-storage
ms.devlang: python
ms.topic: how-to
ms.date: 03/31/2021
ms.author: ikivanc
# Customer intent: "As a software developer, I want to run automated tests against Azure Blob Storage using a local emulator, so that I can efficiently validate my code without relying on a cloud environment."
---

# Run automated tests by using Azurite

Learn how to write automated tests against private endpoints for Azure Blob Storage by using the Azurite storage emulator.

## Run tests on your local machine

1. Install the latest version of [Python](https://www.python.org/)

1. Install [Azure Storage Explorer](https://azure.microsoft.com/features/storage-explorer/)

1. Install and run [Azurite](../common/storage-use-azurite.md):

   **Option 1:** Use npm to install, then run Azurite locally

   ```bash
   # Install Azurite
   npm install -g azurite

   # Create an Azurite directory
   mkdir c:\azurite

   # Launch Azurite locally
   azurite --silent --location c:\azurite --debug c:\azurite\debug.log
   ```

   **Option 2:** Use Docker to run Azurite

   ```bash
   docker run -p 10000:10000 mcr.microsoft.com/azure-storage/azurite azurite-blob --blobHost 0.0.0.0
   ```

1. In Azure Storage Explorer, select **Attach to a local emulator**

    :::image type="content" source="media/use-azurite-to-run-automated-tests/blob-storage-connection.png" alt-text="Screenshot of Azure Storage Explorer connecting to Azure Storage source.":::

1. Provide a **Display name** and **Blobs port** number to connect Azurite and use Azure Storage Explorer to manage local blob storage.

   :::image type="content" source="media/use-azurite-to-run-automated-tests/blob-storage-connection-attach.png" alt-text="Screenshot of Azure Storage Explorer attaching to a local emulator.":::

1. Create a virtual Python environment

   ```bash
   python -m venv .venv
   ```

1. Create a container and initialize environment variables. Use a [PyTest](https://docs.pytest.org/) [conftest.py](https://docs.pytest.org/en/latest/how-to/writing_plugins.html#conftest-py-plugins) file to generate tests. Here is an example of a conftest.py file:

   ```python
   from azure.storage.blob import BlobServiceClient
   import os

   def pytest_generate_tests(metafunc):
      os.environ['AZURE_STORAGE_CONNECTION_STRING'] = 'DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=http://127.0.0.1:10000/devstoreaccount1;'
      os.environ['STORAGE_CONTAINER'] = 'test-container'

      # Create a container for Azurite for the first run
      blob_service_client = BlobServiceClient.from_connection_string(os.environ.get("AZURE_STORAGE_CONNECTION_STRING"))
      try:
         blob_service_client.create_container(os.environ.get("STORAGE_CONTAINER"))
      except Exception as e:
         print(e)
   ```

   > [!NOTE]
   > The value shown for `AZURE_STORAGE_CONNECTION_STRING` is the default value for Azurite, it's not a private key.

1. Install dependencies listed in a [requirements.txt](https://github.com/Azure-Samples/automated-testing-with-azurite/blob/main/requirements.txt) file

   ```bash
   pip install -r requirements.txt
   ```

1. Run tests:

   ```bash
   python -m pytest ./tests
   ```

After running tests, you can see the files in Azurite blob storage by using Azure Storage Explorer.

:::image type="content" source="media/use-azurite-to-run-automated-tests/http-local-blob-storage.png" alt-text="Screenshot of Azure Storage Explorer showing files generated by the tests.":::

## Run tests on Azure Pipelines

After running tests locally, make sure the tests pass on [Azure Pipelines](/azure/devops/pipelines). Use a Docker Azurite image as a hosted agent on Azure, or use npm to install Azurite. The following Azure Pipelines example uses npm to install Azurite.

```yaml
trigger:
- master

steps:
- task: UsePythonVersion@0
  displayName: 'Use Python 3.7'
  inputs:
    versionSpec: 3.7

- bash: |
    pip install -r requirements_tests.txt
  displayName: 'Setup requirements for tests'

- bash: |
    sudo npm install -g azurite
    sudo mkdir azurite
    sudo azurite --silent --location azurite --debug azurite\debug.log &
  displayName: 'Install and Run Azurite'

- bash: |
    python -m pytest --junit-xml=unit_tests_report.xml --cov=tests --cov-report=html --cov-report=xml ./tests
  displayName: 'Run Tests'

- task: PublishCodeCoverageResults@1
  inputs:
    codeCoverageTool: Cobertura
    summaryFileLocation: '$(System.DefaultWorkingDirectory)/**/coverage.xml'
    reportDirectory: '$(System.DefaultWorkingDirectory)/**/htmlcov'

- task: PublishTestResults@2
  inputs:
    testResultsFormat: 'JUnit'
    testResultsFiles: '**/*_tests_report.xml'
    failTaskOnFailedTests: true
```

After running the Azure Pipelines tests, you should see output similar to this:

:::image type="content" source="media/use-azurite-to-run-automated-tests/azure-pipeline.png" alt-text="Screenshot of Azure Pipelines test results.":::

## Next steps

- [Use the Azurite emulator for local Azure Storage development](../common/storage-use-azurite.md)
- [Sample: Using Azurite to run blob storage tests in Azure DevOps Pipeline](https://github.com/Azure-Samples/automated-testing-with-azurite)
