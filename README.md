# HelloApplication

This is a sample containerized aspdotnet core application that can be easily deployed to Azure App Service. Azure App Service has been used for deployment here as it is the best option to publish a single-container application. For multi-container applications, we can publish the same application to Azure Kubernetes service.

Prerequisites for application creation:
1) Visual Studio 2019
2) Docker desktop installed on Windows.

The coding part has been done and the publishing of the application on Azure App Service has also been tested. As I have already exhausted my free tier, so just tested if the application published on Azure App Service is accessible, and then deleted the same. For publishing the application at your end, follow the steps given below:
1) Right-click your project in Solution Explorer and choose Publish.
2) On the publish target dialog, choose App Service Linux or App Service.
3) In the subsequent window, click on Create New and then click on publish.
4) Check that you're signed in with the account that's associated with your Azure subscription, and choose a unique name, subscription, resource group and hosting plan, or accept the defaults.
5) Click on the site link to verify your app works as expected in Azure.
6) The publishing profile is saved with all the details you selected and can be re-used if we want to do the deployments in the future.

The application can be accessed easily by hitting the url that we get for the Azure app service.

The deployment can also be done using Azure DevOps or Jenkins, where we need to create the pipeline for the same. In case of Azure DevOps, we would create a yaml file having the pipeline. I have created one earlier for publishing my application to ACR(Azure Container Registry) and then using the same for deployment of the application on the AKS cluster, but not using the same method here as AKS is costly and that could incur high cost. The yaml file would look like as given below (line 19 to 44):
  
# Build Docker image for this app using Azure Pipelines
# http://docs.microsoft.com/azure/devops/pipelines/languages/docker?view=vsts
pool:
  vmImage: 'Ubuntu 16.04'

variables:
  buildConfiguration: 'Release'
  imageName: 'dotnetcore:$(Build.BuildId)'
  # define two more variables dockerId and dockerPassword in the build pipeline in UI

steps:
- script: |
    dotnet build --configuration $(buildConfiguration)
    dotnet test dotnetcore-tests --configuration $(buildConfiguration) --logger trx
    dotnet publish --configuration $(buildConfiguration) --output out
    docker build -f Dockerfile -t $(dockerId)/$(imageName) .
    docker login -u $(dockerId) -p $pswd
    docker push $(dockerId)/$(imageName)
  env:
    pswd: $(dockerPassword)

- task: PublishTestResults@2
  condition: succeededOrFailed()
  inputs:
    testRunner: VSTest
    testResultsFiles: '**/*.trx'

For this, we would need the relevant variables to be defined and then the automated deployment could be performed.
