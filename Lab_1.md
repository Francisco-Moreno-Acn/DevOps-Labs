# Lab 1 - Create and Run a Pipeline

## Step 1: Create a new DevOps Project

- Login to [Azure DevOps](https://dev.azure.com/)
- Click on + New Project
- Set project name and visibility

## Step 2: Clone the Repository

- Go to Repos Tab
- Select to clone in VS Code
- Create a folder in your local machine to use for this repository
- VS Code prompts you to open the repository, select Open

## Step 3: Install Azure Pipelines Extension

- In VS Code, select View > Extensions or Ctrl+Shift+X and search for Azure Pipelines
- Click on Install

## Step 4: Create a YAML Pipeline Definition

- In VS Code, open the Explorer and hover over your project´s name folder, select New Folder and name it "Deploy"
- Create a new file and name it "azure-pipelines.yml"
- Use the next template as an example for the YAML file:
```
trigger: none

pool:
  vmImage: ubuntu-latest

jobs:
- job:
  steps:
  - script: echo Hello world!
    displayName: 'Placeholder step'"
```

- Open a Bash Terminal
- Commit and Push the YAML file to your repo:
```
git add deploy/azure-pipelines.yml
git commit -m "Add initial pipeline definition"
git push
```

## Step 5: Set Up Azure Pipelines
- Go to Azure DevOps
- Select Pipelines Tab
- Click on Create Pipeline
- On the Connect Tab, select Azure Repos Git
- Choose your repository
- On the Configure your Pipeline pane, select Existing Azure Pipelines YAML file
- Leave the branch as main and set the path as: /deploy/azure-pipelines.yml
- Review your pipeline YAML and click on Run
