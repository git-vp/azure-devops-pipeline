# azure-devops-pipeline

## DevOps pipeline for NodeJs App in Azure App Service
### Create App Service and Web App
To run an Azure DevOps pipeline to build and deploy NodeJs app in Azure App Service, first create an App Service Plan and Azure Web App:
1. Select a default Azure region
   `az configure --defaults location=eastus`
2. Set the following environment variables
   * Assign a random number to generate unique resources
     `resourceSuffix=$RANDOM`
   * Set Azure App Service Web App name
     `webName="helloworld-nodejs-${resourceSuffix}"`
   * Set Resource Group name and Azure App Service Plan name
     `rgName='hello-world-nodejs-rg'`
     `planName='helloworld-nodejs-plan'` 
3. Create a Resource Group
   `az group create --name $rgName`
4. Create an App Service Plan
   `az appservice plan create --name $planName --resource-group $rgName --sku B1 --is-linux`
5. Create an Azure Web App
   `az webapp create --name $webName --resource-group $rgName --plan $planName --runtime "node|16-lts"`
6. Retrieve the web host name (FQDN)
   `az webapp list --resource-group $rgName --query "[].{hostName: defaultHostName, state: state}" --output table`

### Create DevOps Pipeline
An run an Azure DevOps pipeline, you need:
* DevOps Agent - is a compute instance that is either provided by Azure or can be a self hosted agent. To use self hosted agent, see [Self Hosted Agents](https://github.com/git-vp/azure-devops-self-hosted-agents)
* Service Connection - is required for a pipeline to connect to remote services for executing tasks in a pipeline. For example, an ADO agent would establish a connection with Azure Resource Manager (ARM) to create Azure resources.
* Pipeline - is a YAML definition code to define jobs and tasks

1. To create a pipeline, go to DevOps Organisation like https://dev.azure.com/{DevOps-Org-Name}
2. Click on `New Project`, give a project name, and choose version control as `Git`
3. Clone the repo and submit app source code

   `git clone https://{DevOps-Org-Name}@dev.azure.com/{DevOps-Org-Name}/{ProjectName}/_git/{ProjectName}`
   
   `git add .`
   
   `git status`
   
   `git commit -m "Initial commit"`
   
   `git branch -m main`
   
   `git push -u origin main`
   
5. Create a Service Connection
   * Open the project
   * Click on `Project Settings -> Service Connections (under Pipelines)`
   * Click on `New Service Connection`, and select `Azure Resource Manager`, authentication method as `Service principal (automatic)`
   * Scope level is `Subscription`, choose the Subscription, Resource Group, Service connection name and select `Access permission on all pipelines`
6. Now create an Azure pipeline
   * Open the project
   * Under Pipelines, click on `New pipeline`
   * Choose `Azure Repos Git` as your source control
   * Choose the repository in the project you created above
   * In `Configure your pipeline` options, select `Node.js Express Web App to Linux on Azure`
   * Then select Azure Subscription
   * Under `Web App Name` select App Service Web App instance created above
   * Click on `Validate and configure`. This will generate YAML source for your pipeline
   * Save the YAML code
  
7. When you create an Azure pipeline, it will automatically generate a Service Connection. The identifier of this service connection is given under:
   ```yaml
    variables:
  
    # Azure Resource Manager connection created during pipeline creation
    azureSubscription: 'd0f724e8-b94f-4fd5-be89-cb9ce352a3c5'
   ```
   You can change this hexa-decimal value and replace it with the Service Connection you created above
8. To use self hosted ADO agent replace the agent name from:
   ```
     pool:
      vmImage: $(vmImageName)
   ```
   to:
   ```
     pool:
      name: {ado-agents-pool-name}
   ```
9. Save and run the pipeline
