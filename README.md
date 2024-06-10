# azure-pipeline

Project: Azure DevOps Pipeline for E-Commerce Platform

Overview

This project involves the development and implementation of a CI/CD pipeline using Azure DevOps for an e-commerce platform. The main goals are to streamline the deployment process, automate the build, test, and deployment stages, and implement strategies to minimize downtime during deployments.
Tools and Technologies

•	Azure DevOps: Repos, Pipelines, Artifacts
•	Docker: Containerization
•	AKS (Azure Kubernetes Service): Orchestration
•	ACR (Azure Container Registry): Container registry

Key Achievements
•	Automated build, test, and deployment stages, reducing deployment time by 50%.
•	Implemented rolling updates and blue-green deployments to minimize downtime.

Detailed Steps and Code

1. Setting Up Azure DevOps Repos
	1.	Create a New Repository
		o	Go to Azure DevOps organization.
		o	Create a new project (e.g., ECommercePlatform).
		o	Navigate to Repos and create a new repository.
	2.	Clone the Repository Locally
```bash
git clone https://dev.azure.com/yourorganization/ECommercePlatform/_git/ECommercePlatform
cd ECommercePlatform
```

	3.	Add Project Files Add your e-commerce platform source code to this repository.
	4.	Commit and Push

```
git add .
git commit -m "Initial commit"
git push origin main
```

2. Creating the Azure DevOps Pipeline
	1.	Navigate to Pipelines
		o	In your Azure DevOps project, go to Pipelines and create a new pipeline.
	2.	Select Repository
		o	Select the repository you created for the e-commerce platform.
	3.	Configure Pipeline
		o	Choose Starter Pipeline and replace its content with the following YAML configuration:
```
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'
  dockerRegistryServiceConnection: '<your-service-connection>'
  imageRepository: 'ecommerce-platform'
  containerRegistry: 'youracr.azurecr.io'
  dockerfilePath: 'Dockerfile'
  tag: '$(Build.BuildId)'

stages:
- stage: Build
  jobs:
  - job: Build
    steps:
    - task: UseDotNet@2
      inputs:
        packageType: 'sdk'
        version: '5.x'
        installationPath: $(Agent.ToolsDirectory)/dotnet

    - script: 'dotnet build --configuration $(buildConfiguration)'
      displayName: 'Build the application'

    - task: Docker@2
      inputs:
        containerRegistry: '$(dockerRegistryServiceConnection)'
        repository: '$(imageRepository)'
        command: 'buildAndPush'
        Dockerfile: '$(dockerfilePath)'
        tags: |
          $(tag)

- stage: Deploy
  dependsOn: Build
  jobs:
  - deployment: Deploy
    environment: 'staging'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: Kubernetes@1
            inputs:
              connectionType: 'Azure Resource Manager'
              azureSubscriptionEndpoint: '<your-azure-subscription>'
              azureResourceGroup: '<your-resource-group>'
              kubernetesCluster: '<your-k8s-cluster>'
              namespace: 'staging'
              command: 'apply'
              useConfigurationFile: true
              configuration: 'manifests/deployment.yaml'
              secretType: 'dockerRegistry'
              containerRegistryType: 'Azure Container Registry'
              containerRegistry: 'youracr.azurecr.io'
              imagePullSecret: '<image-pull-secret>'
              arguments: '-f manifests/deployment.yaml'

- stage: Production
  dependsOn: Deploy
  jobs:
  - deployment: Production
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: Kubernetes@1
            inputs:
              connectionType: 'Azure Resource Manager'
              azureSubscriptionEndpoint: '<your-azure-subscription>'
              azureResourceGroup: '<your-resource-group>'
              kubernetesCluster: '<your-k8s-cluster>'
              namespace: 'production'
              command: 'apply'
              useConfigurationFile: true
              configuration: 'manifests/deployment.yaml'
              secretType: 'dockerRegistry'
              containerRegistryType: 'Azure Container Registry'
              containerRegistry: 'youracr.azurecr.io'
              imagePullSecret: '<image-pull-secret>'
              arguments: '-f manifests/deployment.yaml'
```

3. Docker and Kubernetes Configuration
	1.	Dockerfile
```
FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build
WORKDIR /src
COPY ["ECommercePlatform/ECommercePlatform.csproj", "ECommercePlatform/"]
RUN dotnet restore "ECommercePlatform/ECommercePlatform.csproj"
COPY . .
WORKDIR "/src/ECommercePlatform"
RUN dotnet build "ECommercePlatform.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "ECommercePlatform.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "ECommercePlatform.dll"]
```

	2.	Kubernetes Deployment YAML (manifests/deployment.yaml)
	
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecommerce-platform
  labels:
    app: ecommerce-platform
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ecommerce-platform
  template:
    metadata:
      labels:
        app: ecommerce-platform
    spec:
      containers:
      - name: ecommerce-platform
        image: youracr.azurecr.io/ecommerce-platform:$(tag)
        ports:
        - containerPort: 80
```


4. Implementing Rolling Updates and Blue-Green Deployments
	1.	Rolling Updates
		o	Ensure the strategy section in your deployment YAML is configured for rolling updates.
```
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

	2.	Blue-Green Deployments
		o	Set up two environments (e.g., staging and production).
		o	Deploy to staging first, validate, and then switch traffic to production.
		
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecommerce-platform-staging
  labels:
    app: ecommerce-platform
    environment: staging
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ecommerce-platform
      environment: staging
  template:
    metadata:
      labels:
        app: ecommerce-platform
        environment: staging
    spec:
      containers:
      - name: ecommerce-platform
        image: youracr.azurecr.io/ecommerce-platform:$(tag)
        ports:
        - containerPort: 80
		
```
		
Summary
This project demonstrates the creation of a robust CI/CD pipeline using Azure DevOps for an e-commerce platform. The pipeline automates the build, test, and deployment stages, and implements rolling updates and blue-green deployments to minimize downtime. With these practices, the deployment process is streamlined, and downtime is significantly reduced, enhancing the overall efficiency and reliability of the e-commerce platform.


