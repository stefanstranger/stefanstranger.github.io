---
layout: post
title: Running Universal Dashboard Docker container in Azure Web Apps for Containers.
categories: [Azure, PowerShell]
tags: [Azure, PowerShell]
---

In my previous <a href="https://stefanstranger.github.io/powershell/2018/10/02/RunningUniversalDashboardinALinuxDockerContainer.html" target="_blank">blog post</a> I explained how to create a Docker container which is servicing an Universal Dashboard web page.

In this blog post I want to explain how you can host this Universal Dashboard Linux Docker container on Azure App Service on Linux.

## Azure App Service on Linux

Azure Web App is a fully managed compute platform that is optimized for hosting websites and web applications. Customers can use App Service on Linux to host web apps natively on Linux for supported application stacks. 

App Service on Linux supports a number of Built-in images in order to increase developer productivity. If the runtime your application requires is not supported in the built-in images, there are instructions on how to build your own Docker image to deploy to Web App for Containers.

And this is exactly what we are trying to achieve here. We want to use our custom Universal Dashboard Docker image in Web App for Containers.

## Requirements

Before we can use our Docker Containerized Universal Dashboard within an Azure Web App for Container we need the following pre-requisites:
1. Azure Resource Group
2. Azure App Service Plan
3. Azure Container Registry
4. Universal Dashboard Docker image stored in Azure Container Registry. 

For more information about above Azure Resources please look at the References section at the end of this blog post.

## Deployment steps

There are different ways to deploy the Azure Resource Group, App Service Plan and Container Registry. In this blog post I'll be using an ARM Template which deploys these required resources.

Example ARM Template for deploying Resource Group, App Service Plan and Container Registry.

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "resourceGroupLocation": {
            "type": "string",
            "defaultValue": "WestEurope"
        },
        "resourceGroupName": {
            "type": "string",
            "metadata": {
                "description": "The name of the Resource Group where to host the Web App Container."
            }
        },
        "appServicePlanName": {
            "type": "string",
            "metadata": {
                "description": "The name of the App Service plan to use for hosting the web app."
            }
        },
        "servicePlanTier": {
            "type": "string",
            "allowedValues": [
                "Basic",
                "Standard"
            ],
            "defaultValue": "Basic",
            "metadata": {
                "description": "Tier for Service Plan"
            }
        },
        "servicePlanSku": {
            "type": "string",
            "allowedValues": [
                "B1",
                "B2",
                "B3",
                "S1",
                "S2",
                "S3"
            ],
            "defaultValue": "B2",
            "metadata": {
                "description": "Size for Service Plan"
            }
        },
        "acrName": {
            "type": "string",
            "metadata": {
                "description": "The name of the Azure Container Registry"
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.Resources/resourceGroups",
            "apiVersion": "2018-05-01",
            "location": "[parameters('resourceGroupLocation')]",
            "name": "[parameters('resourceGroupName')]"
        },
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2017-05-10",
            "name": "appServicePlanDeployment",
            "resourceGroup": "[parameters('resourceGroupName')]",
            "dependsOn": [
                "[resourceId('Microsoft.Resources/resourceGroups/', parameters('resourceGroupName'))]"
            ],
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {},
                    "variables": {},
                    "resources": [
                        {
                            "comments": "This is the App Service Plan deployment",
                            "apiVersion": "2016-09-01",
                            "name": "[parameters('appServicePlanName')]",
                            "type": "Microsoft.Web/serverfarms",
                            "location": "[parameters('resourceGroupLocation')]",
                            "properties": {
                                "name": "[parameters('appServicePlanName')]",
                                "workerSizeId": "1",
                                "reserved": true,
                                "numberOfWorkers": "1",
                                "hostingEnvironment": ""
                            },
                            "sku": {
                                "Tier": "[parameters('servicePlanTier')]",
                                "Name": "[parameters('servicePlanSku')]"
                            },
                            "kind": "linux"
                        },
                        {
                            "type": "Microsoft.ContainerRegistry/registries",
                            "sku": {
                                "name": "Premium",
                                "tier": "Premium"
                            },
                            "name": "[parameters('acrName')]",
                            "apiVersion": "2017-10-01",
                            "location": "[parameters('resourceGroupLocation')]",
                            "scale": null,
                            "properties": {
                                "adminUserEnabled": true
                            }
                        }
                    ]
                }
            }
        }
    ],
    "outputs": {
        "acrNameOutput": {
            "value": "[parameters('acrName')]",
            "type": "string"
        },
        "ResourceGroupOutput": {
            "value": "[parameters('resourceGroupName')]",
            "type": "string"
        }
    }
}
```

You can use the following PowerShell script to deploy the pre-requisites:
```powershell
# Deploy-WebAppContainerPrereqs.ps1
# Use this script to manually deploy the ARM Template with parameter input file.
# Change variables before running deployment

#region Variables
$Location = 'WestEurope' 
$ResourceGroupName = 'WebAppContainer-rg' 
$appServicePlanName = 'UDAppServiceLinuxPlan'
$servicePlanTier = 'Basic'
$servicePlanSku = 'B1'
$acrName = 'UDContainerRegistry'
#endregion

#region Connect to Azure
Add-AzureRmAccount
 
#Select Azure Subscription
$subscription = 
(Get-AzureRmSubscription |
        Out-GridView `
        -Title 'Select an Azure Subscription ...' `
        -PassThru)
 
Set-AzureRmContext -SubscriptionId $subscription.Id -TenantId $subscription.TenantID
#endregion

#region variables
$ARMTemplateFile = 'C:\Users\[username]\Source\Repos\Contoso\Products\WebAppContainer\Templates\WebAppContainerPrereqs.json'
#endregion

#region create ARM Template Parameter object
$parametersARM = @{}
$parametersARM.Add('resourceGroupLocation', $Location)
$parametersARM.Add('resourceGroupName', $ResourceGroupName)
$parametersARM.Add('appServicePlanName', $appServicePlanName)
$parametersARM.Add('servicePlanTier', $servicePlanTier)
$parametersARM.Add('servicePlanSku', $servicePlanSku)
$parametersARM.Add('acrName', $acrName)
#endregion

#region Deploy ARM Template
   
#region Test ARM Template
Test-AzureRmDeployment `
    -TemplateFile $ARMTemplateFile `
    -TemplateParameterObject $parametersARM `
    -Location $Location `
    -OutVariable testarmtemplate
#endregion

#region Deploy ARM Template with local Parameter file
$result = (New-AzureRMDeployment `
        -TemplateFile $ARMTemplateFile `
        -Location $Location `
        -TemplateParameterObject $parametersARM -Verbose -DeploymentDebugLogLevel All)
$result
#endregion

#endregion
```

After deploying the above ARM Template you would see the following Azure Resource Deployed.

![Azure Resources Deployed in Azure Portal](/assets/webapplinux_1.png)

## Build Docker image and Push Docker image to Azure Container Registry

You could use the Azure DevOps Pipelines to build a Docker container and push that container to the Azure Container Registry, but in this blog post we will use the docker commandline tool.

### Build the image from the Docker file
Read my earlier blog post "<a href="https://stefanstranger.github.io/powershell/2018/10/02/RunningUniversalDashboardinALinuxDockerContainer.html" target="_blank">Running Universal Dashboard in a Linux Docker Container</a>" to see how you can build a (Linux) Docker image with an example Universal Dashboard.

![Docker Build command screenshot](/assets/webapplinux_5.png)

Result:
![Result of Docker Build Command](/assets/webapplinux_6.png)

### Push the Docker image to Azure Container Registry

#### Log in to Azure Container Registry
In order to push an image to the registry, you need to supply credentials so the registry accepts the push. You can retrieve these credentials by using the following PowerShell command.

```powershell
Get-AzureRmContainerRegistryCredential -ResourceGroupName $ResourceGroupName -Name $acrName
```
![Container Registry Credential Screenshot](/assets/webapplinux_2.png)

Retrieve Login Server name of Azure Container Registry:
```powershell
Get-AzureRmContainerRegistry -ResourceGroupName $ResourceGroupName -Name $acrName | Select-Object Name, LoginServer, ResourceGroupName
```

![Container Registry properties screenshot](/assets/webapplinux_3.png)

From your local terminal window, log in to the Azure Container Registry using the docker login command. The server name is required to log in. Use the format azure-container-registry-name>.azurecr.io. Type in your password into the console at the prompt.

```bash
docker login udcontainerregistry.azurecr.io --username UDContainerRegistry
```
![login to azure container registry screenshot](/assets/webapplinux_4.png)

#### Push an image to Azure Container Registry

Push the image by using the docker push command. Tag the image with the name of the registry, followed by your image name and tag.

If you're using your own image, tag the image as follows:
```bash
docker tag udcontainerregistry.azurecr.io/udhelloworld
```
The tag was already added during the build of the docker image step.

![tag docker image screenshot](/assets/webapplinux_7.png)

**Remark:**

This can take some time before the push to the Azure Container Registry is finished. So get yourself a cup of coffee or go for lunch and wait for the push to be finished. If you do this step in your Azure DevOps Pipeline is way faster in my experience.

![Docker images overview screenshot](/assets/webapplinux_8.png)

![Azure Portal ACS Repository screenshot](/assets/webapplinux_9.png)

PowerShell script to push the docker image to the ACR:
```powershell
# PushDockerImage.ps1
# Use this script to manually Push the Docker image to the ACR.
# Change variables before running deployment

#region Variables
$ResourceGroupName = 'WebAppContainer-rg' 
$acrName = 'UDContainerRegistry'
$dockerimagename = 'udhelloworld'
$dockerimageversion = 'latest'
$dockerfile = 'C:\Users\[username]\Source\Repos\Contoso\Products\WebAppContainer\Dockerfile\dockerfile'
#endregion

#region Connect to Azure
Add-AzureRmAccount
 
#Select Azure Subscription
$subscription = 
(Get-AzureRmSubscription |
        Out-GridView `
        -Title 'Select an Azure Subscription ...' `
        -PassThru)
 
Set-AzureRmContext -SubscriptionId $subscription.Id -TenantId $subscription.TenantID
#endregion

#region Build the image from the Docker file
# docker images are case-sensitive. Make sure acrname is lowercase.
$command = ('Get-Content -Path $dockerfile | docker build - -t {0}.azurecr.io/{1}:{2}' -f $($acrName.ToLower()), $dockerimagename, $dockerimageversion)
Invoke-Expression $command
#endregion

#region Push the Docker image to Azure Container Registry
#Log in to Azure Container Registry
$ContainerAdminUser = Get-AzureRmContainerRegistryCredential -ResourceGroupName $ResourceGroupName -Name $acrName

#Retrieve Login Server name of Azure Container Registry
Get-AzureRmContainerRegistry -ResourceGroupName $ResourceGroupName -Name $acrName | Select-Object Name, LoginServer, ResourceGroupName

#login in Docker Container Registry
$command = ('docker login {0}.azurecr.io --username {1} --password {2}' -f $($acrName.ToLower()), $ContainerAdminUser.Username, $ContainerAdminUser.Password)
Invoke-Expression $command

#Push Docker image to ACR
$command = ('docker push {0}.azurecr.io/{1}:{2}' -f $($acrName.ToLower()), $dockerimagename, $dockerimageversion)
Invoke-Expression $command
#endregion
```


#### Deploy and Configure Web App to use the image from Azure Container Registry

In the last step we are going to deploy and configure the Azure Web App using below ARM Template.


ARM Template:
```json
{
	"$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
	"contentVersion": "1.0.0.0",
	"parameters": {
		"resourceGroupLocation": {
			"type": "string",
			"defaultValue": "WestEurope",
			"metadata": {
				"description": "The name of the Resource Group location."
			}
		},
		"resourceGroupName": {
			"type": "string",
			"metadata": {
				"description": "The name of the Resource Group."
			}
		},
		"appServicePlanName": {
			"type": "string",
			"metadata": {
				"description": "The name of the App Service plan to use for hosting the web app."
			}
		},
		"acrName": {
			"type": "string",
			"metadata": {
				"description": "The Name of the Azure Container Registry"
			}
		},
		"siteName": {
			"type": "string",
			"metadata": {
				"description": "Name of azure web app"
			}
		},
		"acrImageName": {
			"type": "string",
			"metadata": {
				"description": "The name of the docker registry"
			}
		},
		"acrUserName": {
			"type": "string",
			"metadata": {
				"description": "User name of the docker registry"
			}
		},
		"acrUserPassword": {
			"type": "string",
			"metadata": {
				"description": "The password of the docker registry"
			}
		},
		"port": {
			"type": "string",
			"defaultValue": "8585",
			"metadata": {
				"description": "The port of the Azure container Universal Dashboard Container."
			}
		},
		"hostNameSslStates": {
			"type": "array",
			"defaultValue": []
		}
	},
	"variables": {
		"hostingPlanName": "[parameters('appServicePlanName')]",
		"acrUrl": "[tolower(concat(parameters('acrName'), '.azurecr.io'))]",
		"linuxFxVersion": "[concat('DOCKER|',variables('acrurl'), '/', parameters('acrImageName'), ':latest')]"
	},
	"resources": [
		{
			"type": "Microsoft.Resources/resourceGroups",
			"apiVersion": "2018-05-01",
			"location": "[parameters('resourceGroupLocation')]",
			"name": "[parameters('resourceGroupName')]"
		},
		{
			"type": "Microsoft.Resources/deployments",
			"apiVersion": "2017-05-10",
			"name": "WebAppContainerDeployment",
			"resourceGroup": "[parameters('resourceGroupName')]",
			"dependsOn": [
				"[resourceId('Microsoft.Resources/resourceGroups/', parameters('resourceGroupName'))]"
			],
			"properties": {
				"mode": "Incremental",
				"template": {
					"$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
					"contentVersion": "1.0.0.0",
					"parameters": {},
					"variables": {},
					"resources": [
						{
							"comments": "This is the Linux web app with the Docker image from Azure Container Registry",
							"type": "Microsoft.Web/sites",
							"kind": "app,linux,container",
							"name": "[parameters('siteName')]",
							"apiVersion": "2016-03-01",
							"location": "[parameters('resourceGroupLocation')]",
							"properties": {
								"name": "[parameters('siteName')]",
								"serverFarmId": "[variables('hostingPlanName')]",
								"hostingEnvironment": "",
								"clientAffinityEnabled": true,
								"httpsOnly": true,
								"siteConfig": {
									"AlwaysOn": true,
									"minTlsVersion": "1.2",
									"ftpsState": "Disabled",
									"appCommandLine": "",
									"linuxFxVersion": "[variables('linuxFxVersion')]"
								},
								"hostNameSslStates": "[parameters('hostNameSslStates')]"
							},
							"resources": [
								{
									"name": "appsettings",
									"type": "config",
									"apiVersion": "2015-08-01",
									"dependsOn": [
										"[resourceId('Microsoft.Web/sites', parameters('siteName'))]"
									],
									"properties": {
										"DOCKER_REGISTRY_SERVER_URL": "[variables('acrUrl')]",
										"DOCKER_REGISTRY_SERVER_USERNAME": "[parameters('acrUserName')]",
										"DOCKER_REGISTRY_SERVER_PASSWORD": "[parameters('acrUserPassword')]",
										"WEBSITES_PORT": "[parameters('port')]"
									}
								}
							]
						},
						{
							"type": "Microsoft.Web/sites/hostNameBindings",
							"name": "[concat(parameters('siteName'), '/' , if(empty(parameters('hostNameSslStates')),'dummy',parameters('hostNameSslStates')[copyIndex()].name))]",
							"condition": "[not(empty(parameters('hostNameSslStates')))]",
							"apiVersion": "2016-03-01",
							"location": "[parameters('resourceGroupLocation')]",
							"properties": {},
							"dependsOn": [
								"[parameters('siteName')]"
							],
							"copy": {
								"name": "bindingsCopy",
								"count": "[if(empty(parameters('hostNameSslStates')),1,length(parameters('hostNameSslStates')))]",
								"mode": "serial",
								"batchSize": 1
							}
						}
					]
				}
			}
		}
	],
	"outputs": {
		"imageNameOutput": {
			"value": "[parameters('acrImageName')]",
			"type": "string"
		},
		"dockerRegistryUrlOutput": {
			"value": "[variables('acrUrl')]",
			"type": "string"
		},
		"linuxFxVersionOutput": {
			"value": "[variables('linuxFxVersion')]",
			"type": "string"
		}
	}
}
```

With the following PowerShell script we can deploy the ARM Template:
```powershell
# Deploy-WebAppContainer.ps1
# Use this script to manually deploy the ARM Template with parameter input file.
# Change variables before running deployment

#region Variables
$Location = 'WestEurope' 
$ResourceGroupName = 'WebAppContainer-rg' 
$appServicePlanName = 'UDAppServiceLinuxPlan'
$acrName = 'UDContainerRegistry'
$siteName = 'udhelloworlddemo'
$acrImageName = 'udhelloworld'
#endregion

#region Connect to Azure
Add-AzureRmAccount
 
#Select Azure Subscription
$subscription = 
(Get-AzureRmSubscription |
        Out-GridView `
        -Title 'Select an Azure Subscription ...' `
        -PassThru)
 
Set-AzureRmContext -SubscriptionId $subscription.Id -TenantId $subscription.TenantID
#endregion

#region variables
$ARMTemplateFile = 'C:\Users\[username]\Source\Repos\Contoso\Products\WebAppContainer\Templates\WebAppContainer.json'
#endregion

#region get ACR Admin user credentials
$ContainerAdminUser = Get-AzureRmContainerRegistryCredential -ResourceGroupName $ResourceGroupName -Name $acrName
#endregion

#region create ARM Template Parameter object
$parametersARM = @{}
$parametersARM.Add('resourceGroupLocation', $Location)
$parametersARM.Add('resourceGroupName', $ResourceGroupName)
$parametersARM.Add('appServicePlanName', $appServicePlanName)
$parametersARM.Add('acrName', $acrName)
$parametersARM.Add('siteName', $siteName)
$parametersARM.Add('acrUserName', $ContainerAdminUser.Username)
$parametersARM.Add('acrUserPassword', $ContainerAdminUser.Password)
$parametersARM.Add('acrImageName', $acrImageName)
#endregion

#region Deploy ARM Template
   
#region Test ARM Template
Test-AzureRmDeployment `
    -TemplateFile $ARMTemplateFile `
    -TemplateParameterObject $parametersARM `
    -Location $Location `
    -OutVariable testarmtemplate
#endregion

#region Deploy ARM Template with local Parameter file
$result = (New-AzureRMDeployment `
        -TemplateFile $ARMTemplateFile `
        -Location $Location `
        -TemplateParameterObject $parametersARM -Verbose -DeploymentDebugLogLevel All)
$result
#endregion

#endregion
```

Result:
![Azure Portal Resources deployed screenshot](/assets/webapplinux_10.png)


Hope you learned something new with this blog post.


**References:**
* <a href="https://stefanstranger.github.io/powershell/2018/10/02/RunningUniversalDashboardinALinuxDockerContainer.html" target="_blank">Blog post - Running Universal Dashboard in a Linux Docker Container</a>
* <a href="https://docs.microsoft.com/en-us/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview" target="_blank">Azure App Service Plan</a>
* <a href="https://azure.microsoft.com/en-us/services/container-registry/" target="_blank">Azure Container Registry</a>
* <a href="https://docs.microsoft.com/en-us/azure/devops/pipelines/?view=vsts" target="_blank">Azure DevOps Pipelines</a>
* <a href="https://docs.microsoft.com/en-us/azure/app-service/containers/tutorial-custom-docker-image" target="_blank">Use a custom Docker image for Web App for Containers</a>
* <a href="https://stefanstranger.github.io/powershell/2018/10/02/RunningUniversalDashboardinALinuxDockerContainer.html" target="_blank">Running Universal Dashboard in a Linux Docker Container</a>
* <a href="https://github.com/stefanstranger/Azure/tree/master/azure-templates/uddashboard-web-app-container" target="_blank">Github repo containing used ARM Templates and scripts</a>