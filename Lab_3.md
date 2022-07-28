# Add a Bicep Deployment Task to the Pipeline

## Step 1: Add your Website´s Bicep File to the Git Repository

- Open VS Code Explorer
- In the "Deploy" folder, create a new file named main.bicep
- Copy the following code into the main.bicep file:
```
@minLength(3)
@maxLength(24)
param storageName string ='storage${uniqueString(resourceGroup().id)}'
param webAppName string = 'webapp${uniqueString(resourceGroup().id)}'
param sku string = 'F1'
param location string = 'eastus'
var appServicePlanName = 'appplan${uniqueString(resourceGroup().id)}'

resource ExampleStorageDemo 'Microsoft.Storage/storageAccounts@2021-02-01' = {
  name: storageName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}

resource appServicePlan 'Microsoft.Web/serverfarms@2020-12-01' = {
  name: appServicePlanName
  location: location
  sku: {
    name: sku
  }
}

resource appService 'Microsoft.Web/sites@2020-06-01' = {
  name: webAppName
  location: location
  properties: {
    serverFarmId: appServicePlan.id
  }
}

output webappnameid string = webAppName 

```
- In the VS Code terminal, run the following commands:
```
git add deploy/main.bicep
git commit -m 'Add Bicep file'
git push
```

## Step 2: Adding "WebApp" Code for Azure Deployment

- Before having any succesful run, you must add an app for the purpose of versioning. The CI/CD Pipeline will deploy the latest version of an app 
- Go to the "Microsoft Demo App" and copy all files from the https://github.com/MicrosoftDocs/mslearn-tailspin-spacegame-web repo
- After having all the files in your local repo, you must commit the changes to the remote repo

> When you have all files at your current repo, you may proceed and make changes to the app, in this case the easiest way to "modify" this "Microsoft Learning Demo App" is to go to the Home Page file location and make a change on the headers. The file is located in the next path TailSpin.SpaceGame.Web/Views/Home/Index.cshtml.

- This is the cshtml Home Page example, you may modify. You're most likely to make a change on the main titles of the app, where we have the "`<p>App Version 3 from Latest DevOps Pipeline</p>`" and "`<h2>DevOps Leaderboard</h2>`" app titles.    

```
@model TailSpin.SpaceGame.Web.Models.LeaderboardViewModel
@{
    ViewData["Title"] = "Home Page";
}
<section class="intro">
    <div class="container">
        <img class="title" src="/images/space-game-title.svg" alt="Space Game">
        <p>App Version 3 from Latest DevOps Pipeline</p>
    </div>
</section>
<section class="download">
    <div class="image-cap"></div>
    <div class="container">
        <a href="" class="btn btn-default btn-lg" data-toggle="modal" data-target="#pretend-modal">Download game</a>
    </div>
</section>

<!-- Screenshots -->
<section class="screens">
    <div class="container">
        <ul>
            <li><a href="" data-toggle="modal" data-target=".pic-01"><img src="/images/space-game-placeholder.svg" alt=""></a></li>
            <li><a href="" data-toggle="modal" data-target=".pic-01"><img src="/images/space-game-placeholder.svg" alt=""></a></li>
            <li><a href="" data-toggle="modal" data-target=".pic-01"><img src="/images/space-game-placeholder.svg" alt=""></a></li>
            <li><a href="" data-toggle="modal" data-target=".pic-01"><img src="/images/space-game-placeholder.svg" alt=""></a></li>
        </ul>
    </div>
</section>

<!-- Leaderboard -->
<section class="leaderboard">
    <div class="container">
        <a name="leaderboard"></a>
        <h2>DevOps Leaderboard</h2>
```
> When all the new changes or features are applied to the code follow the same process to push the code as in the previous stages 

- In the VS Code terminal, run the following commands:
```
git add . 
git commit -m 'Add New CHanges/Features to the App'
git push

```

## Step 3: Replace the Pipeline Steps

- In VS Code, open the deploy/azure-pipelines.yml file
- Copy the following code into the azure-pipelines.yml file 
```
trigger: 
  branches:
    include: 
    - master

pool: Self-Hosted

variables: 
  buildConfiguration: 'Release' 

  resourceGroupName: 'PipelineDevOpsLab'
  azureServiceConnection: 'AzureConnection'
  deployment: 'deployment1'
  templateFile: 'main.bicep'
  location: 'eastus'

stages:

- stage: build_project 
  displayName: "Build Project"
  
  jobs:
    - job: "building_project"
      displayName: "Build Project" 
      steps: 
      - task: UseDotNet@2 
        displayName: 'Use .NET SDK 6.x' 
        inputs: 
          packageType: sdk 
          version: '6.x' 

      - task: Npm@1 
        displayName: 'Run npm install' 
        inputs: 
          verbose: false 

      - script: './node_modules/.bin/node-sass Tailspin.SpaceGame.Web/wwwroot --output Tailspin.SpaceGame.Web/wwwroot' 
        displayName: 'Compile Sass assets' 

      - task: gulp@1 
        displayName: 'Run gulp tasks' 

      - script: 'echo "$(Build.DefinitionName), $(Build.BuildId), $(Build.BuildNumber)" > buildinfo.txt' 
        displayName: 'Write build info' 
        workingDirectory: Tailspin.SpaceGame.Web/wwwroot 

      - task: DotNetCoreCLI@2 
        displayName: 'Restore project dependencies' 
        inputs: 
          command: 'restore' 
          projects: '**/*.csproj' 

      - task: DotNetCoreCLI@2 
        displayName: 'Build the project - Release' 

        inputs: 
          command: 'build' 
          arguments: '--no-restore --configuration Release' 
          projects: '**/*.csproj' 

      - task: DotNetCoreCLI@2
        displayName: 'Publish the project - $(buildConfiguration)'
        inputs:
          command: 'publish'
          projects: '**/*.csproj'
          publishWebProjects: false
          arguments: '--no-build --configuration $(buildConfiguration) --output  $(Pipeline.Workspace)/$(buildConfiguration)'
          zipAfterPublish: true

- stage: publish_artifact 
  displayName: "Publish Artifact"
  jobs:
    - job: "publishing_artifact"
      displayName: "Publish Artifact" 

      steps: 
      - task: PublishBuildArtifacts@1
        displayName: 'Publish Artifact: WebApp'
        inputs:
          artifactName: 'WebApp' 
          pathToPublish: $(Pipeline.Workspace)/$(buildConfiguration)

- stage: download_artifact 
  displayName: "Download Artifact"
  jobs:
    - job: "downloading_artifact"
      displayName: "Download Artifact" 

      steps:
      - task: DownloadPipelineArtifact@2
        displayName: Download Artifact
        inputs:
          source: 'current'
          artifactName: 'WebApp'
          itemPattern: '**/*.zip'
          path: $(Pipeline.Workspace)/$(buildConfiguration)

- stage: infrastructure_zip_deployment
  displayName: "Zip Deployment - Az Infra"
  jobs:
    - job: "deploying_infra_and_zip"
      displayName: "Zip Deployment -Az Infra" 
    
      steps:
      - task: AzureCLI@2
        displayName: 'Azure Infrastructure Deployment'
        inputs:
          azureSubscription: $(azureServiceConnection)
          scriptType: ps
          scriptLocation: inlineScript
          inlineScript: |
            az group create --name $(resourceGroupName) --location $(location)
            az deployment group create --name $(deployment) --resource-group $(resourceGroupName) --template-file $(templateFile)
            $outputwebappname = az deployment group show -g $(resourceGroupName) -n $(deployment) --query properties.outputs.webappnameid.value
            $webappname = $outputwebappname -replace '"',''
            Write-Host "##vso[task.setvariable variable=webappsetvariable]$webappname"

      - task: AzureWebApp@1
        inputs:
          azureSubscription: $(azureServiceConnection)
          appName: $(webappsetvariable)
          package: '$(Pipeline.Workspace)/$(buildConfiguration)/*.zip'
```
- In the VS Code terminal, run the following commands:
```
git add deploy/azure-pipelines.yml
git commit -m 'Add Updated Yaml File'
git push

```
- After having all set with the main code for the Yaml and Bicep files, the Pipeline created in the previous Lab is ready to go for some testing 

## Step 4: Verify the Azure Portal to check the New WebApp Version deployed 

- Head to the Azure portal: https://azure.microsoft.com/en-us/get-started/azure-portal/
- Go to the top right corner, click on the sign in box and log in to your account 
- Once you've sign in you must search for the resource group with the search bar at the top middle of the azure portal homepage screen 
- After you found the resource group you'll be able to see alll three resources created from the Pipeline Run (Storage Account, App Service and App Service Plan)

> Note: After Step 3 is done, the Pipeline will automatically trigger after any commit.   
