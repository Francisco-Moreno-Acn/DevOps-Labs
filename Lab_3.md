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

@description('Indicates whether to deploy the storage account for toy manuals.')
param deployToyManualsStorageAccount bool

@description('A unique suffix to add to resource names that need to be globally unique.')
@maxLength(13)
param resourceNameSuffix string = uniqueString(resourceGroup().id)

var appServiceAppName = 'toy-website-${resourceNameSuffix}'
var appServicePlanName = 'toy-website-plan'
var toyManualsStorageAccountName = 'toyweb${resourceNameSuffix}'

// Define the SKUs for each component based on the environment type.
var environmentConfigurationMap = {
  nonprod: {
    appServicePlan: {
      sku: {
        name: 'F1'
        capacity: 1
      }
    }
    toyManualsStorageAccount: {
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
    toyManualsStorageAccount: {
      sku: {
        name: 'Standard_ZRS'
      }
    }
  }
}
var toyManualsStorageAccountConnectionString = deployToyManualsStorageAccount ? 'DefaultEndpointsProtocol=https;AccountName=${toyManualsStorageAccount.name};EndpointSuffix=${environment().suffixes.storage};AccountKey=${toyManualsStorageAccount.listKeys().keys[0].value}' : ''

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
          name: 'ToyManualsStorageAccountConnectionString'
          value: toyManualsStorageAccountConnectionString
        }
      ]
    }
  }
}

resource toyManualsStorageAccount 'Microsoft.Storage/storageAccounts@2021-02-01' = if (deployToyManualsStorageAccount) {
  name: toyManualsStorageAccountName
  location: resourceGroup().location
  kind: 'StorageV2'
  sku: environmentConfigurationMap[environmentType].toyManualsStorageAccount.sku
}
```
- In the VS Code terminal, run the following commands:
```
git add deploy/main.bicep
git commit -m 'Add Bicep file'
git push
```

## Step 2: Replace the Pipeline Steps

- In VS Code, open the deploy/azure-pipelines.yml file
- 
