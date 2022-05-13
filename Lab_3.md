# Add a Bicep Deployment Task to the Pipeline

## Step 1: Add your Website´s Bicep File to the Git Repository

- Open VS Code Explorer
- In the "Deploy" folder, create a new file named main.bicep
- Copy the following code into the main.bicep file:
```
@description('The Azure region into which the resources should be deployed.')
param location string = resourceGroup().location

@description('The type of environment. This must be nonprod or prod.')
@allowed([
  'nonprod'
  'prod'
])
param environmentType string

@description('Indicates whether to deploy the storage account.')
param deployLiveDemoStorageAccount bool

@description('A unique suffix to add to resource names that need to be globally unique.')
@maxLength(13)
param resourceNameSuffix string = uniqueString(resourceGroup().id)

var appServiceAppName = 'live-demo-website-${resourceNameSuffix}'
var appServicePlanName = 'live-demo-website-plan'
var liveDemoStorageAccountName = 'live-demo${resourceNameSuffix}'

// Define the SKUs for each component based on the environment type.
var environmentConfigurationMap = {
  nonprod: {
    appServicePlan: {
      sku: {
        name: 'F1'
        capacity: 1
      }
    }
    liveDemoStorageAccount: {
      sku: {
        name: 'Standard_LRS'
      }
    }
  }
  prod: {
    appServicePlan: {
      sku: {
        name: 'S1'
        capacity: 2
      }
    }
    liveDemoStorageAccount: {
      sku: {
        name: 'Standard_ZRS'
      }
    }
  }
}
var liveDemoStorageAccountConnectionString = deployLiveDemoStorageAccount ? 'DefaultEndpointsProtocol=https;AccountName=${liveDemoStorageAccount.name};EndpointSuffix=${environment().suffixes.storage};AccountKey=${liveDemoStorageAccount.listKeys().keys[0].value}' : ''

resource appServicePlan 'Microsoft.Web/serverFarms@2020-06-01' = {
  name: appServicePlanName
  location: location
  sku: environmentConfigurationMap[environmentType].appServicePlan.sku
}

resource appServiceApp 'Microsoft.Web/sites@2020-06-01' = {
  name: appServiceAppName
  location: location
  properties: {
    serverFarmId: appServicePlan.id
    httpsOnly: true
    siteConfig: {
      appSettings: [
        {
          name: 'LiveDemoStorageAccountConnectionString'
          value: liveDemoStorageAccountConnectionString
        }
      ]
    }
  }
}

resource liveDemoStorageAccount 'Microsoft.Storage/storageAccounts@2021-02-01' = if (deployLiveDemoStorageAccount) {
  name: liveDemoStorageAccountName
  location: resourceGroup().location
  kind: 'StorageV2'
  sku: environmentConfigurationMap[environmentType].liveDemoStorageAccount.sku
}
```
- In the VS Code terminal, run the following commands:
```
git add deploy/main.bicep
git commit -m "Add Bicep file"
git push
```

## Step 2: Replace the Pipeline Steps

- In VS Code, open the deploy/azure-pipelines.yml file
- Remove the last 2 lines in your YAML file and add a task that uses the az deployment group create command:
```
trigger: none

pool:
  vmImage: ubuntu-latest

jobs:
- job:
  steps:
  
  - task: AzureCLI@2
    inputs:
      azureSubscription: $(ServiceConnectionName)
      scriptType: 'bash'
      scriptLocation: 'inlineScript'
      inlineScript: |
        az deployment group create \
          --name $(Build.BuildNumber) \
          --resource-group $(ResourceGroupName) \
          --template-file deploy/main.bicep \
          --parameters environmentType=$(EnvironmentType) deployLiveDemoStorageAccount=$(DeployLiveDemoStorageAccount)
```
- in VS Code, run:
```
git add deploy/azure-pipelines.yml
git commit -m 'Add Azure CLI tasks to pipeline'
git push
```
## Step 3: Add Pipeline Variables

- In [Azure DevOps](https://dev.azure.com/) , go to Pipelines tab
- Select your Pipeline and click on Edit
- Select Variables and click on New Variable
- use the following Variables:
  - 
