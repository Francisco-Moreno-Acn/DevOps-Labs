# Create a Service Connection

## Step 1: Sign in to Azure

- In VS Code, run the following Azure CLI command:
```
az login
```

## Step 2: Create a Resource Group in Azure

- In VS Code, run the following Azure CLI command:
```
az group create --name PipelineDevOpsLab --location eastus
```

## Step 3: Create a Service Connection in Azure Pipelines

- In your DevOps Project, select ⚙️Project Settings 
- Under Pipelines, select Service Connections
- Click on Create Service Connection
- Select Azure Resource Manager
- Select Service Principal (automatic)
- Select scope level as Subscription and set your Subscription, Resource Group and provide a Service Connection Name
- [x] Grant Access Permission to all Pipelines and Save
- In Service Connections, verify that your new service connection is shown
