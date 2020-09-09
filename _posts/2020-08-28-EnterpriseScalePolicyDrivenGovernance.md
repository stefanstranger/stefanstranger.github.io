---
layout: post
title: Enterprise-Scale - Policy Driven Governance
categories: [Azure]
tags: [Azure]
comments: true
---

In this second blog post in the blog series about becoming an Enterprise-Scale Subject Matter Expert I want to share what I did to better understand the Enterprise-Scale design principle <u>Policy Driven Governance</u>.

- [Documentation](#documentation)
- [What does policy-driven governance mean?](#what-does-policy-driven-governance-mean)
  - [Azure Policy in the context of Enterprise-Scale](#azure-policy-in-the-context-of-enterprise-scale)
- [Enterprise-Scale reference implementation Wingtip](#enterprise-scale-reference-implementation-wingtip)
  - [Enterprise-Scale "in-a-box"](#enterprise-scale-in-a-box)
  - [Deploy Reference implementation](#deploy-reference-implementation)
- [AzGovViz - Azure Governance Visualizer](#azgovviz---azure-governance-visualizer)
- [Getting started with infrastructure-as-code](#getting-started-with-infrastructure-as-code)
  - [AzOps](#azops)
    - [Discovery](#discovery)
    - [Deployment](#deployment)
      - [How are changes in the compliance-as-code artifacts pushed to Azure?](#how-are-changes-in-the-compliance-as-code-artifacts-pushed-to-azure)
      - [How are Policies Assigned using AzOps?](#how-are-policies-assigned-using-azops)
        - [Create a policyAssignment.parameters.json file](#create-a-policyassignmentparametersjson-file)
        - [Create policyAssignment into the management group file](#create-policyassignment-into-the-management-group-file)
        - [Can you remove a Policy using AzOps?](#can-you-remove-a-policy-using-azops)
        - [How are the AzState parameters files being used to declare and deploy ARM Resources?](#how-are-the-azstate-parameters-files-being-used-to-declare-and-deploy-arm-resources)
    - [Does AzOps support Azure DevOps?](#does-azops-support-azure-devops)
    - [How does AzOps Github Action works?](#how-does-azops-github-action-works)
- [Turn requirements into code example](#turn-requirements-into-code-example)
  - [Compliant](#compliant)
  - [Enterprise-Scale reference Policies](#enterprise-scale-reference-policies)
  - [Deploy Key Vault](#deploy-key-vault)
- [References](#references)

Read below documentation to get started.

# Documentation

* Azure Architecture Blog - Enterprise-Scale and Azure Policy for policy-driven governance [1]

* Microsoft Docs - What is Azure Policy [2]

* Github- Azure Policy Examples [3]

* Microsoft Docs - Get Compliancy data of Azure resources [4]

* Microsoft Docs - Understand Policy effects [5]

* Microsoft Docs - How compliancy works [6]

* Microsoft Docs - Order of Evaluation [7]

* Github Enterprise-Scale - AzOpsReference [8]

* Github Enterprise-Scale - Deploy Enterprise-Scale Reference implementation in your own environment [9]

* Linkedin - PlatformOps in a Microsoft Enterprise-scale landing zone [19]

# What does policy-driven governance mean?

Policy-driven governance means the usage of Azure Policy to build and provide <u>guardrails</u>, and to enable <u>autonomy</u> for the platform and application teams, regardless of their scale points. Those guardrails ensure that deployed workloads and applications are <u>compliant</u> with your organization’s security and compliance requirements, and therefore a secure path to the public cloud. [1]

## Azure Policy in the context of Enterprise-Scale

As outlined in the Enterprise-Scale design principles, Azure Policy is used to build and provide the required guardrails for all landing zones [15]. 
Enterprise-Scale is primarily focusing on <u>proactive</u> and <u>preventive policies</u> (e.g. with DeployIfNotExists, or in short DINE) to enable autonomy for the platform, autonomy for the application teams, and ensures that resources are in their compliant goal state, <u>no matter how those resources got created</u>. [1]

Enterprise-Scale includes at the moment three reference implementations (<a href="https://github.com/Azure/Enterprise-Scale/blob/main/docs/reference/contoso/Readme.md" target="_blank">Contoso</a>, <a href="https://github.com/Azure/Enterprise-Scale/blob/main/docs/reference/adventureworks/README.md" target="_blank">AdventureWorks</a> and <a href="https://github.com/Azure/Enterprise-Scale/blob/main/docs/reference/wingtip/README.md" target="_blank">Wingtip</a>) to help simplify the adoption of those proactive and preventive policies.

I choose to deploy the Wingtip Enterprise-Scale reference implementation because this is an Azure-only example.


# Enterprise-Scale reference implementation Wingtip 

This reference implementation is ideal for customers who want to start with Landing Zones for their workload in Azure, where hybrid connectivity to their on-premise datacenter is not required from the start. [10]

## Enterprise-Scale "in-a-box"

For the deployment of the Wingtip Enterprise-Scale Reference architecture I used the Enterprise-Scale "in-a-box" solution. This helps deploy the Enterprise-Scale 1st party reference implementation using a single (I used two subscriptions) Azure subscription for training and evaluation purposes only. [11]

I followed the steps described in the documentation on Github. [11]

Overview of environment used for the deployment of Enterprise-Scale "in-a-box"

**Azure Subscriptions**:

| Subscription Name | Tenant | Offer |
|----------|----------|----------|
| Landing Zone | sstranger.onmicrosoft.com | MSDN |
| Management | sstranger.onmicrosoft.com | MSDN |


**Github Repository:**

| Github Repository name | Github Account | Visibility|
|----------|----------|----------|
| ES-IAB | stefanstranger | Private |

During the deployment I encountered some issues which were solved by below solutions.

| Issue    | Solutions |
|----------|----------|
| There are optional deployments for Wingtips reference deployment, like enabling policy-driven governance for your landing zone(s). Deployment will fail if you only have one Subscription, e.g. for Management| Create another Subscription (Landing Zone) besides your initial (Management Subscription. |
| Deployment failed with error: "The subscription '629ff310-8477-4ffc-8a85-2555d56c6912' is not registered to use microsoft.insights. | First validated that Microsoft.Insights was a registered Resource Provider on mentioned Subscription, which was the case. A redeployment resulted in a successful deployment |

Overview of Wingtip Azure Portal deployment configuration.

![Azure Portal deployment overview](/assets//WingtipDeploymentOverview.png)

The reference implementations and the templates, are designed and optimized for Azure Portal deployment (Deploy to Azure). The current ARM templates being used for the Enterprise-Scale reference bootstrapped implementation heavily rely on Azure Portal Deployment and it’s deployment is a onetime activity. Once the platform resources are deployed, management of the Enterprise-Scale Landing Zone resources will happen via Infrastructure-as-code and changes are deployed in a controlled manner using a CI/CD pipeline in Azure DevOps or Github.

Overview of the bootstrapped Enterprise-Scale reference implementation using CloudSkew [12].

![Wingtip Azure Architecture overview](/assets//Enterprise-ScaleArchitecture.png)

## Deploy Reference implementation

The deployment of the reference implementations (e.g Wingtip, Contoso or AdventureWorks) contains of a bootstrapped deployed of the Management Groups, Resources and Builtin/Custom Policies/Initiatives/Assignments on selected reference implementation infrastructure.

For more information about what is being deployed please look at the Wingtip reference documentation. [25]

The reference implementation deployments are run using the Deploy to Azure Portal Deployment. 

![Wingtip customized deployment](/assets//WingtipCustomDeployment.png)

![](/assets//CustomDeploymentscreen.png)

When the Deploy to Azure button is selected a <u>Custom Deployment Blade</u> is triggered which calls the <a href="https://github.com/Azure/Enterprise-Scale/blob/main/docs/reference/wingtip/armTemplates/es-foundation.json" target="_blank">es-foundation.json</a> ARM Template with a custom Azure Portal UI defined by the <a href="https://github.com/Azure/Enterprise-Scale/blob/main/docs/reference/wingtip/armTemplates/portal-es-foundation.json" target="_blank">portal-es-foundation.json</a> [26]

```json
...
"steps": [
            {
                "name": "lzSettings",
                "label": "Enterprise-Scale Company prefix",
                "subLabel": {
                    "preValidation": "Provide a company prefix for the management group structure that will be created.",
                    "postValidation": "Done"
                },
                "bladeTitle": "Company prefix",
                "elements": [
                    {
                        "name": "infoBox1",
                        "type": "Microsoft.Common.InfoBox",
                        "visible": true,
                        "options": {
                            "icon": "Info",
                            "text": "Enterprise-Scale ARM deployment requires access at the tenant root (/) scope. Visit this link to ensure you have the appropriate RBAC permission to complete the deployment",
                            "uri": "https://docs.microsoft.com/azure/role-based-access-control/elevate-access-global-admin"
                        }
                    },
                    {
                        "name": "esMgmtGroup",
                        "type": "Microsoft.Common.TextBox",
                        "label": "Management Group prefix",
                        "toolTip": "Provide a prefix for management group structure, 1-5 characters.",
                        "defaultValue": "",
                        "constraints": {
                            "required": true,
                            "regex": "^[a-z0-9A-Z-]{1,5}$",
                            "validationMessage": "The prefix must be 1-5 characters."
                        }
                    }
                ]
            }
...
```

Based on the configuration made in the Custom Deployment Blade, resource deployments are triggered stored in the auxilery folder of the Enterprise-Scale Github repository.

<img src="/assets//AuxiliaryFolder.png" alt="AuxileryFolderw" width="733" height="447"/>


# AzGovViz - Azure Governance Visualizer

After the deployment of the above Azure infrastructure I wanted to see the configured Azure Governance capabilities such as as Azure Policy, RBAC etc.

**Tip:**

Julian Hayward created an Azure Governance Visualizer called AzGovViz that shows exactly what I was looking for. [20]

![](/assets//AzGovViz.png)

# Getting started with infrastructure-as-code

Now we have configured the Wingtip reference implementation it's time to configure the requirements to enable infrastructure-as-code for the Wingtip Azure architecture.

I followed the steps described at the Github Enterprise-Scale Getting started with infrastructure-as-code pages.[13]

The most interesting step after the Github CICD setup is the <u>discovery</u> of your environment using the <u>Github Action AzOps</u>. [14]

The AzOps GitHub Action  will create the following artifacts in your GitHub repository:

* Current Management Group, Subscriptions, Policy Definitions and Policy Assignments are discovered, and RESTful representation of the resources are saved as ARM Template parameters file.


* It will create system branch representing your current configuration as ARM template parameter file and merge it automatically into main.

After reading the LinkedIn post from Anders Bonde called PlatformOps in a Microsoft Enterprise-scale landing zone [19] it's time to investigate <u>"Compliance-as-Code"</u>. This will enable the documentation of the compliance in code. This can show the regulators what actually has been implemented to be compliant using the Policy-driven Governance. This is where <u>AzOps</u> comes into play.

## AzOps

The AzOps GitHub Action is rooted in the principle that **Everything in Azure** is a resource and to operate at-scale, it should be managed **declaratively to determine target goal state** of the overall platform. [14]

With AzOps you are able to document compliance in code. But it offers more:
![](/assets//AzOps-implementation-scope.png)

### Discovery
Before starting the Enterprise-Scale journey, it is important for customers to discover existing configuration in Azure that can serve as platform baseline. Consequence of not discovering existing environment will be no reference point to rollback or roll-forward after deployment. Discovery is also important for organizations, who are starting their DevOps and Infrastructure-as-code (IaC) journey, as this can provide crucial on-ramp path to allow transitioning without starting all-over. [14]

For the purpose of discovery, following resources are considered within the scope of overall Azure platform. This will initialize the Git repository with current configuration to baseline configuration encompassing following:

* Management Group hierarchy and Subscription organization
    * ResourceTypes:
        * Microsoft.Management/managementGroups
        * Microsoft.Management/managementGroups/subscriptions
        * Microsoft.Subscription/subscriptions
* Policy Definition and Policy Assignment for Governance
    * ResourceTypes:
        * Microsoft.Authorization/policyDefinitions
        * Microsoft.Authorization/policySetDefinitions
        * Microsoft.Authorization/policyAssignments
* Role Definition and Role Assignment
    * ResourceTypes:
        * Microsoft.Authorization/roleDefinitions
        * Microsoft.Authorization/roleAssignments

| Note | 
|----------|
| It's highly recommended to read the documentation for the AzOps Github Action [14]| 

### Deployment

The deployment component of AzOps allows for deploying templates to the Azure environment using the AzOps-Push pipeline by committing templates at the appropriate scope without the need for extra deployment scripts. One pipeline to rule them all.

#### How are changes in the compliance-as-code artifacts pushed to Azure?

Now we understand how the existing Azure environment is pulled down (compliance-as-code), how are changes made, reflected in Azure?

For this we can use the guidance on the Github Enterprise-Scale Github repository, called "Deploy your own ARM templates with AzOps GitHub Actions" [21]

After following the steps to enforce a resource naming convention the Github Workflow AzOps-Push should have picked up the change being made.

Below the part of the Github Workflow log where the change is detected and using ARM as orchestrator.

```PowerShell
2020-08-25T14:24:20.9612237Z [14:24:20.9605] (git) Fetching latest origin changes
2020-08-25T14:24:21.2880880Z From https://github.com/stefanstranger/ES-IAB
2020-08-25T14:24:21.2881507Z  * [new branch]      NamingConvention -> origin/NamingConvention
2020-08-25T14:24:21.2882036Z  * [new branch]      main             -> origin/main
2020-08-25T14:24:21.4107052Z [14:24:21.4098] (git) Invoking AzOps Change
2020-08-25T14:24:21.4181905Z [14:24:21.4175] (git) Deployment required
2020-08-25T14:24:21.4234183Z [14:24:21.4228] (git) Add / Modify:
2020-08-25T14:24:21.4246418Z [14:24:21.4240] (git) azops/Tenant Root Group (496f0b27-4fa4-4c3d-8bbe-19c4b6875c81)/ES (ES)/policyDef-NamingConvention.json
2020-08-25T14:24:21.4252488Z [14:24:21.4247] (git) azops/Tenant Root Group (496f0b27-4fa4-4c3d-8bbe-19c4b6875c81)/ES (ES)/policyDef-NamingConvention.parameters.json
2020-08-25T14:24:21.4257776Z [14:24:21.4253] (git) Delete:
2020-08-25T14:24:21.8595229Z [14:24:21.8587] (pwsh) Template Parameter found /github/workspace/azops/Tenant Root Group (496f0b27-4fa4-4c3d-8bbe-19c4b6875c81)/ES (ES)/policyDef-NamingConvention.parameters.json
2020-08-25T14:24:22.1946139Z [14:24:22.1938] (pwsh) Template found /github/workspace/azops/Tenant Root Group (496f0b27-4fa4-4c3d-8bbe-19c4b6875c81)/ES (ES)/policyDef-NamingConvention.json
2020-08-25T14:24:31.1570134Z 
2020-08-25T14:24:31.1578100Z Id                      : /providers/Microsoft.Management/managementGroups/ES/p
2020-08-25T14:24:31.1579206Z                           roviders/Microsoft.Resources/deployments/AzOps-policy
2020-08-25T14:24:31.1579785Z                           Def-NamingConvention
2020-08-25T14:24:31.1580327Z DeploymentName          : AzOps-policyDef-NamingConvention
2020-08-25T14:24:31.1580656Z ManagementGroupId       : ES
2020-08-25T14:24:31.1580965Z Location                : northeurope
2020-08-25T14:24:31.1581280Z ProvisioningState       : Succeeded
2020-08-25T14:24:31.1581608Z Timestamp               : 8/25/2020 2:24:26 PM
2020-08-25T14:24:31.1581918Z Mode                    : Incremental
2020-08-25T14:24:31.1582223Z TemplateLink            : 
2020-08-25T14:24:31.1582517Z Parameters              : 
2020-08-25T14:24:31.1582858Z                           Name                 Type                       
2020-08-25T14:24:31.1583183Z                           Value     
2020-08-25T14:24:31.1583527Z                           ===================  =========================  
2020-08-25T14:24:31.1583860Z                           ==========
2020-08-25T14:24:31.1584200Z                           policyName           String                     
2020-08-25T14:24:31.1584909Z                           enforce-resource-naming
2020-08-25T14:24:31.1585379Z                           policyDescription    String                     
2020-08-25T14:24:31.1585748Z                           Policy to enforce naming convention pattern.
2020-08-25T14:24:31.1586127Z                           namePattern          String                     es*  
2020-08-25T14:24:31.1586459Z                                
2020-08-25T14:24:31.1586758Z                           
2020-08-25T14:24:31.1587043Z Outputs                 : 
2020-08-25T14:24:31.1587331Z DeploymentDebugLogLevel : 
2020-08-25T14:24:31.1587493Z 
```

Searching the AzOps Github repository I found the following scripts that are used to push the changes to Azure:
* <u><a href="https://github.com/Azure/AzOps/blob/main/src/private/Invoke-AzOpsChange.ps1" target="_blank">Invoke-AzOpsChange.ps1</a></u>

  This cmdlet processes AzOps changes

* <u><a href="https://github.com/Azure/AzOps/blob/main/src/private/New-AzOpsDeployment.ps1" target="_blank">New-AzOpsDeployment.ps1</a></u>

  This cmdlet processes AzOpsState changes and takes appropriate action by invoking ARM deployment and limited set of imperative operations required by platform that is currently not supported in ARM
  
 
Let's test what happens when we create two new files (policyDef-NamingConvention.json and policyDef-NamingConvention.parameters.json) and use the New-AzOpsDeployment.ps1 script to update the Azure environment.

policyDef-NamingConvention.json

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/managementGroupDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "policyName": {
            "type": "string",
            "metadata": {
                "description": "Provide name for the policyDefinition."
            }
        },
        "policyDescription": {
            "type": "string",
            "metadata": {
                "description": "Provide a description for the policy."
            }
        },
        "namePattern": {
            "type": "string",
            "metadata": {
                "description": "Provide naming pattern."
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.Authorization/policyDefinitions",
            "apiVersion": "2019-09-01",
            "name": "[parameters('policyName')]",
            "properties": {
                "description": "[parameters('policyDescription')]",
                "displayName": "[parameters('policyName')]",
                "policyRule": {
                    "if": {
                        "not": {
                            "field": "name",
                            "like": "[parameters('namePattern')]"
                        }
                    },
                    "then": {
                        "effect": "deny"
                    }
                }
            }
        }
      ]
    }
```

policyDef-NamingConvention.parameters.json

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
       "policyName": {
           "value": "enforce-resource-naming"
       },
       "policyDescription": {
           "value": "Policy to enforce naming convention pattern."
       },
       "namePattern": {
           "value": "es*"
       }
     }

```

```PowerShell
<#
    Because the example to create a Naming Convention Policy on Management Group scope the code in the New-AzDeployment Function uses the New-AzManagementGroupDeployment cmdlet.
    Below code will deploy the policy definition at the ES management group scope.
    Link: https://github.com/Azure/Enterprise-Scale/blob/main/docs/Deploy/deploy-new-arm.md#how-to-deploy-arm-templates-with-azops
#>

#region Deploy Naming Convention Policy at Management Group 'ES' Scope
$parameters = @{
    'TemplateFile'                = 'C:\Users\stefstr\Documents\GitHub\ES-IAB\azops\Tenant Root Group (496f0b27-4fa4-4c3d-8bbe-19c4b6875c81)\ES (ES)\policyDef-NamingConvention.json'
    'TemplateParameterFile'       =  'C:\Users\stefstr\Documents\GitHub\ES-IAB\azops\Tenant Root Group (496f0b27-4fa4-4c3d-8bbe-19c4b6875c81)\ES (ES)\policyDef-NamingConvention.parameters.json'
    'Location'                    = 'westeurope'
    'ManagementGroupId'           = 'ES'
    'SkipTemplateParameterPrompt' = $true
    'Debug'                       = $true
}

New-AzManagementGroupDeployment @parameters
#endregion
```

So we now understand how (custom) Policy Definitions can be deployed using AzOps, but how does AzOps assigns this Policy?

| Note | 
|----------|
| Committing changes to Enterprise-Scale Github repository are automatically handled by the AzOps CICD pipelines. There no need to manually deploy these changes. Above was just an example showing how ARM templates for Custom Policy Definition are normally deployed. | 


#### How are Policies Assigned using AzOps?

To create a Policy Assignment using AzOps, there are three ways to do so:

1. Create a policyAssignment.parameters.json file
   Similar to the ones in the azopsreference. AzOps will pick it up and deploy it using ARM template [24]
   
2. Create policyAssignment into the management group file (e.g. Microsoft.Management_managementGroups-ES.parameters.json).
   AzOps will do the same as above, and if the effect is deployIfNotExist, it will also create subsequent roleAssignment for that policy at scope. Further, if the policyRule is targeting Microsoft.Resources/subscriptions, AzOps will also remediate the subscription as part of the policyAssignment process.
   
3. Last but not least, you can bring your own template doing all of the above, where the template includes policyAssignment, roleAssignment, and subsequent deployment resources.

On the Enterprise-Scale Github repository there is documentation regarding the deployment of Policy Assignments. [22]

##### Create a policyAssignment.parameters.json file

To assign the earlier created and deployed Naming Convention Policy Definition we can create the following policyAssignment.parameters.json file and store that under the .AzState folder:

```batch
Tenant Root Group (496f0b27-4fa4-4c3d-8bbe-19c4b6875c81)
    ├───.AzState
    └───ES (ES)
        ├───.AzState
```


Microsoft.Authorization_policyAssignments-Enforce-Naming-Convention.parameters.json

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "input": {
        "value": {
          "Identity": null,
          "Location": null,
          "Name": "Enf-Naming-Convention",
          "PolicyAssignmentId": "/providers/Microsoft.Management/managementGroups/ES/providers/Microsoft.Authorization/policyAssignments/Enf-Naming-Convention",
          "Properties": {
            "Description": "Assigning Enforce Naming Convention using  policyAssignment.parameters.json",
            "DisplayName": "enforce-resource-naming",
            "NotScopes": null,
            "Parameters": {},
            "PolicyDefinitionId": "/providers/Microsoft.Management/managementGroups/ES/providers/Microsoft.Authorization/policyDefinitions/enforce-resource-naming",
            "Scope": "/providers/Microsoft.Management/managementGroups/ES"
          },
          "ResourceId": "/providers/Microsoft.Management/managementGroups/ES/providers/Microsoft.Authorization/policyAssignments/Enf-Naming-Conventionn",
          "ResourceName": "Enf-Naming-Convention",
          "ResourceType": "Microsoft.Authorization/policyAssignments",
          "Sku": {
            "name": "A0",
            "tier": "Free"
          }
        }
      }
    }
  }
```

Remark:

* Length of Policy Assignment Name cannot exceed 24 characters

Create branch, make changes, commit and push, create PR, approve and merge PR. After the AzOps Push Github Workflow the Policy Assignment was executed.

![Policy Assignment in Azure Portal](/assets//PolicyAssignment.png)

##### Create policyAssignment into the management group file 

First we need to remove the enforce-resource-naming Policy assignment, before trying out the second option to assign a Policy. I used the Azure Portal to remove this Policy Assignment.

![Delete Policy Assignment](/assets//deleteAssignment.png)

After the deletion of the Policy Assignment I manually triggered the AzOps-Pull Workflow to retrieve the latest compliance-as-code Azure environment state. The initial Microsoft.Authorization_policyAssignments-Enforce-Naming-Convention.parameters.json file should be removed from your Github repository.

Each Microsoft.Management-managementGroups_<managementgroupscope>.parameters.json file has the following section, and it is the most important part of the parameter files and the section you have to primarily apply changes to using this guide. You can learn more about the *.parameters.json schema here. [23]

```json
# Empty part of a parameter JSON file after initialization
"properties": {
    "policySetDefinitions": [],
    "roleAssignments": [],
    "policyAssignments": [],
    "policyDefinitions": [],
    "roleDefinitions": []
}
```

We now need to add below code to the Microsoft.Management_managementGroups-ES.parameters.json file:

```json
"policyAssignments": [
            {
              "Identity": null,
              "Location": null,
              "Name": "Enf-Naming-Convention",
              "PolicyAssignmentId": "/providers/Microsoft.Management/managementGroups/ES/providers/Microsoft.Authorization/policyAssignments/Enf-Naming-Convention",
              "Properties": {
                "Description": "Assigning Enforce Naming Convention using managementgroup.parameters.json",
                "DisplayName": "enforce-resource-naming",
                "NotScopes": null,
                "Parameters": {},
                "PolicyDefinitionId": "/providers/Microsoft.Management/managementGroups/ES/providers/Microsoft.Authorization/policyDefinitions/enforce-resource-naming",
                "Scope": "/providers/Microsoft.Management/managementGroups/ES"
              },
              "ResourceId": "/providers/Microsoft.Management/managementGroups/ES/providers/Microsoft.Authorization/policyAssignments/Enf-Naming-Conventionn",
              "ResourceName": "Enf-Naming-Convention",
              "ResourceType": "Microsoft.Authorization/policyAssignments",
              "Sku": {
                "name": "A0",
                "tier": "Free"
              }
            }
          ]
```

Create branch, make changes, commit and push, create PR, approve and merge PR. After the AzOps Push Github Workflow the Policy Assignment was executed.

![Screenshot of Azure Policy Assignment using managementgroup.parameters.json](/assets//PolicyAssignment-option2.png)

When the AzOps-Pull Github Workflow is run you see that the Microsoft.Management_managementGroups-ES.parameters.json file in the main branch now contains above Policy Assignment code and that a new Microsoft.Authorization_policyAssignments-Enf-Naming-Convention.parameters.json file is created with the following content:

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "input": {
      "value": {
        "Identity": null,
        "Location": "eastus",
        "Name": "Enf-Naming-Convention",
        "PolicyAssignmentId": "/providers/Microsoft.Management/managementGroups/ES/providers/Microsoft.Authorization/policyAssignments/Enf-Naming-Convention",
        "Properties": {
          "Description": "Assigning Enforce Naming Convention using managementgroup.parameters.json",
          "DisplayName": "enforce-resource-naming",
          "NotScopes": null,
          "Parameters": {},
          "PolicyDefinitionId": "/providers/Microsoft.Management/managementGroups/ES/providers/Microsoft.Authorization/policyDefinitions/enforce-resource-naming",
          "Scope": "/providers/Microsoft.Management/managementGroups/ES"
        },
        "ResourceId": "/providers/Microsoft.Management/managementGroups/ES/providers/Microsoft.Authorization/policyAssignments/Enf-Naming-Convention",
        "ResourceName": "Enf-Naming-Convention",
        "ResourceType": "Microsoft.Authorization/policyAssignments",
        "Sku": {
          "name": "A0",
          "tier": "Free"
        }
      }
    }
  }
}

```

##### Can you remove a Policy using AzOps?

Suppose I want to remove the both the Policy Assignment and Policy Definition (enforce-naming-convention) can I then use AzOps?

I removed the Policy Assignment info in the Microsoft.Management_managementGroups-ES.parameters.json and also removed the Microsoft.Authorization_policyAssignments-Enf-Naming-Convention.parameters.json file. And again followed the steps: create branch, make changes, commit and pushe, create PR, approve and merge PR in main branch.


![screenshot removed assignment files](/assets//remove-assignment-files.png)

After the AzOps-Push workflow finized I was still able to see the Policy Assignment.

![](/assets//PolicyAssignment-still-present.png)

Let's see what happens when I manually trigger the AzOps-Pull Github Workflow. It turns out that the Microsoft.Authorization_policyAssignments-Enf-Naming-Convention.parameters.json is recreated. That is to be expected because the Azure Policy Assignment was still present.

And the Microsoft.Management_managementGroups-ES.parameters.json again contained the following Policy Assignment information:

```json
{
              "Identity": null,
              "Location": "eastus",
              "Name": "Enf-Naming-Convention",
              "PolicyAssignmentId": "/providers/Microsoft.Management/managementGroups/ES/providers/Microsoft.Authorization/policyAssignments/Enf-Naming-Convention",
              "Properties": {
                "Description": "Assigning Enforce Naming Convention using  managegementgroup.parameters.json",
                "DisplayName": "enforce-resource-naming",
                "NotScopes": null,
                "Parameters": {},
                "PolicyDefinitionId": "/providers/Microsoft.Management/managementGroups/ES/providers/Microsoft.Authorization/policyDefinitions/enforce-resource-naming",
                "Scope": "/providers/Microsoft.Management/managementGroups/ES"
              },
              "ResourceId": "/providers/Microsoft.Management/managementGroups/ES/providers/Microsoft.Authorization/policyAssignments/Enf-Naming-Convention",
              "ResourceName": "Enf-Naming-Convention",
              "ResourceType": "Microsoft.Authorization/policyAssignments",
              "Sku": {
                "name": "A0",
                "tier": "Free"
              }
            }
```
**Conclusion:** 

You cannot use AzOps to delete Policy Assignments and Azure Policy Definitions.

##### How are the AzState parameters files being used to declare and deploy ARM Resources?

According to the Enterprise-Scale Schema documentation [23] an <a href="https://github.com/Azure/AzOps/blob/main/template/tenant.json" target="_blank">Enterprise-Scale ARM template (tenant.json) </a> is being used.

Let's see if we can create a new Policy Assignment using the <a href="https://github.com/Azure/AzOps/blob/main/src/private/New-AzOpsDeployment.ps1#L137" target="_blank">New-AzManagementGroupDeployment</a> Function within the New-AzOpsDeployment.ps1 script of the AzOps Github Action PowerShell Module.

We need to use the <u>New-AzManagementGroupDeployment</u> cmdlet because the enforce-resource-naming Assignment needs to be assigned on the ManagementGroup Scope ES.

First we need to delete the enforce-resource-naming Policy Assignment to be able to check if below PowerShell script will Assign the Policy on ManagementGroup Scope ES. After deletion of the enforce-resource-naming Policy Assignment, we can run below PowerShel script to re-assign the Policy using the tenant.json Enterprise-Scale ARM template.

```PowerShell
<#
    Assign enforce-naming-convention Azure Policy on Management Group Scope ES with the Enterprise-Scale tenant.json ARM template.

    Download raw content of https://raw.githubusercontent.com/Azure/AzOps/main/template/tenant.json to local development machine
#>

$parameters = @{
    'TemplateFile'                = 'C:\temp\tenant.json'
    'TemplateParameterFile'       = 'C:\Users\stefstr\Documents\GitHub\ES-IAB\azops\Tenant Root Group (496f0b27-4fa4-4c3d-8bbe-19c4b6875c81)\ES (ES)\.AzState\Microsoft.Authorization_policyAssignments-Enf-Naming-Convention.parameters.json'
    'Location'                    = 'westeurope'
    'ManagementGroupId'           = 'ES'
    'SkipTemplateParameterPrompt' = $true
    'Debug'                       = $true
}
New-AzManagementGroupDeployment @parameters

```

![Screenshot tenant.json template deployment](/assets//New-AzManagementGroupDeployment-using-tenant-json.png)

### Does AzOps support Azure DevOps?

Most of my customers are using Azure DevOps, so I wanted to know if AzOps supports Azure DevOps.

Currently support for Azure DevOps is in preview. [16]

**Tip:**

Just as for the Github Workflow you need to configure the Credentials for the Azure DevOps Pull and Push Pipelines. When configuring the AZURE_CREDENTIAL secret variable make sure that the JSON string has the double quotes escaped with a backslash, e.g. " becomes \\"

The following PowerShell code helps create the AZURE_CREDENTIALS string input as required

```PowerShell
$AZURE_CREDENTIALS = @'
{
    "clientId": "71561b48-073f-4918-b909-0add97186892",
    "displayName": "es-stefstr",
    "name": "http://es-stefstr",
    "clientSecret": "9f338a83-cf39-49cb-aa49-d069776d895e",
    "tenantId": "15c24bb3-2687-480d-9cbc-fee4ef10cc21",
    "subscriptionId": "d2328fd0-0982-4e43-97ea-798c2c555661"
  } 
'@ | convertfrom-json 

($AZURE_CREDENTIALS | ConvertTo-JSON -Compress) -replace '"','\"'
```
Copy and paste the output to newly created AZURE_CREDENTIALS variable.

<img src="/assets//ADOVariableSecret.png" alt="Azure DevOps AZURE_CREDENTIALS variable" width="309" height="581"/>

### How does AzOps Github Action works?

The AzOps Github Action uses a Docker image [17]

<img src="/assets//GithubAction.png" alt="Github Workflow" width="378" height="462"/>

The <a href="https://github.com/Azure/AzOps/blob/main/entrypoint.ps1" target="_blank">entrypoint</a> PowerShell script being used by the Docker container contains some information which can be used to further investigate how this Github Action works.

Example:

By enabling the Environment variables VERBOSE and DEBUG in the Github Pull Workflow we already can see more information.

<img src="/assets//azops-pull-change.png" alt="Github Pull Workflow" width="460" height="422"/>

To better understand AzOps a good start is to clone the Github AzOps Repository. [14]

I also tried to run the Function Initialize-AzOpsRepository on my local development machine after cloning the Github AzOps repository with the following commands:

```
# Import AzOps PowerShell Module from the location where you cloned the AzOps Github Repository
Import-Module .\src\AzOps.psd1

# If you want to override the location where the AzOpsRepository stores the files representing the existing Azure Environment set the $pwd variable. E.g. $pwd = 'c:\temp'

# Initializes the environment and global variables variables required for the AzOps cmdlets.
Initialize-AzOpsGlobalVariables -Verbose

# This cmdlet initializes the azops repository and takes a snapshot of the entire Azure environment from MG all the way down to resource level.
Initialize-AzOpsRepository -Verbose -SkipResourceGroup -Force

```

This results into the creation of azops-folder in my local repo with all azure resources reflected. We now have compliance-as-code by having a local representation of the existing Azure environment as is.

<img src="/assets//Initialize-AzOpsRepository.png" alt="Initialize AzOps Repository" width="475" height="405"/>

Remark:
* I don't think running the AzOps module functions outside the Github Action is supported.


# Turn requirements into code example

To use Enterprise-Scale to turn requirements (controls) into code you first need to define what you need to be compliant.

Let's try to go from defining compliance to implementation of Compliance-as-Code.

| Azure Service | Identified Risk | Control | Azure Policy DisplayName | Policy Type |
|---------|----------|----------|----------|----------|
| Azure Key Vault | Loss of service level persists longer than needed caused by lack of insight into what went wrong resulting in Downtime | Enable audit logging | <a href="https://www.azadvertizer.net/azpolicyadvertizer/bef3f64c-5290-43b7-85b0-9b254eef4c47.html" target="_blank">Deploy Diagnostic Settings for Key Vault to Log Analytics workspace</a> | Builtin |

**Tip:**

Use <u>AzAdvertizer</u> [24] to provide overview and insights on new releases and changes/updates for Azure Governance capabilities such as Azure Policy, Policy Initiatives, Policy Aliases and RBAC (Role-Based Access Control) Roles.

With above we want to prevent prolonged loss of service for Azure KeyVault by lack of insights for this service. 

With this Azure builtin Policy the diagnostic settings for Key Vault to stream to a central Log Analytics workspace when any Key Vault which is missing this diagnostic settings is created or updated will be deployed.

The following is logged:

* All authenticated REST API requests, including failed requests as a result of access permissions, system errors, or bad requests.
* Operations on the key vault itself, including creation, deletion, setting key vault access policies, and updating key vault attributes such as tags.
* Operations on keys and secrets in the key vault, including:
    * Creating, modifying, or deleting these keys or secrets.
    * Signing, verifying, encrypting, decrypting, wrapping and unwrapping keys, getting secrets, and listing keys and secrets (and their versions).
* Unauthenticated requests that result in a 401 response. Examples are requests that don't have a bearer token, that are malformed or expired, or that have an invalid token.

## Compliant

By showing that for each deployed Key Vault the diagnostic settings for Key Vault is logging to the (central) Log Analytics workspace compliance is achieved.

## Enterprise-Scale reference Policies

If we check the out-of-the-box Enterprise-Scale reference Policies from Wingtip reference Architecture we find the following 
in the ManagementGroup.parameters.json:

```json
{
              "Name": "Deploy-Diagnostics-KeyVault",
              "ResourceId": "/providers/Microsoft.Management/managementGroups/ES/providers/Microsoft.Authorization/policyDefinitions/Deploy-Diagnostics-KeyVault",
              "ResourceName": "Deploy-Diagnostics-KeyVault",
              "ResourceType": "Microsoft.Authorization/policyDefinitions",
              "SubscriptionId": null,
              "PolicyDefinitionId": "/providers/Microsoft.Management/managementGroups/ES/providers/Microsoft.Authorization/policyDefinitions/Deploy-Diagnostics-KeyVault",
              "Properties": {
                "Description": "Apply diagnostic settings for Key Vaults - Log Analytics",
                "DisplayName": "Deploy-Diagnostics-KeyVault",
                "Mode": "All",
                "Parameters": {
                  "logAnalytics": {
                    "type": "String",
                    "metadata": {
                      "displayName": "Log Analytics workspace",
                      "description": "Select the Log Analytics workspace from dropdown list",
                      "strongType": "omsWorkspace"
                    }
                  }
                },
                "PolicyRule": {
                  "if": {
                    "field": "type",
                    "equals": "Microsoft.KeyVault/vaults"
                  },
                  "then": {
                    "effect": "deployIfNotExists",
                    "details": {
                      "type": "Microsoft.Insights/diagnosticSettings",
                      "existenceCondition": {
                        "allOf": [
                          {
                            "field": "Microsoft.Insights/diagnosticSettings/logs.enabled",
                            "equals": "true"
                          },
                          {
                            "field": "Microsoft.Insights/diagnosticSettings/metrics.enabled",
                            "equals": "true"
                          },
                          {
                            "field": "Microsoft.Insights/diagnosticSettings/workspaceId",
                            "equals": "[parameters('logAnalytics')]"
                          }
                        ]
                      },
                      "name": "setByPolicy",
                      "roleDefinitionIds": [
                        "/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"
                      ],
                      "deployment": {
                        "properties": {
                          "mode": "incremental",
                          "template": {
                            "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                            "contentVersion": "1.0.0.0",
                            "parameters": {
                              "resourceName": {
                                "type": "string"
                              },
                              "logAnalytics": {
                                "type": "string"
                              },
                              "location": {
                                "type": "string"
                              }
                            },
                            "variables": {},
                            "resources": [
                              {
                                "type": "Microsoft.KeyVault/vaults/providers/diagnosticSettings",
                                "apiVersion": "2017-05-01-preview",
                                "name": "[concat(parameters('resourceName'), '/', 'Microsoft.Insights/setByPolicy')]",
                                "location": "[parameters('location')]",
                                "dependsOn": [],
                                "properties": {
                                  "workspaceId": "[parameters('logAnalytics')]",
                                  "metrics": [
                                    {
                                      "category": "AllMetrics",
                                      "enabled": true,
                                      "retentionPolicy": {
                                        "days": 0,
                                        "enabled": false
                                      },
                                      "timeGrain": null
                                    }
                                  ],
                                  "logs": [
                                    {
                                      "category": "AuditEvent",
                                      "enabled": true
                                    }
                                  ]
                                }
                              }
                            ],
                            "outputs": {}
                          },
                          "parameters": {
                            "logAnalytics": {
                              "value": "[parameters('logAnalytics')]"
                            },
                            "location": {
                              "value": "[field('location')]"
                            },
                            "resourceName": {
                              "value": "[field('name')]"
                            }
                          }
                        }
                      }
                    }
                  }
                }
              }
            },
```

Within the ManagementGroups.parameter.json file there is a Policy Initiative called <u>Deploy-Diag-LogAnalytics</u>

```json
{
              "Name": "Deploy-Diag-LogAnalytics",
              "PolicySetDefinitionId": "/providers/Microsoft.Management/managementGroups/ES/providers/Microsoft.Authorization/policySetDefinitions/Deploy-Diag-LogAnalytics",
              "Properties": {
                "Description": "This initiative configures application Azure resources to forward diagnostic logs and metrics to an Azure Log Analytics workspace.",
                "DisplayName": "Deploy-Diag-LogAnalytics",
                "Parameters": {
                  "logAnalytics": {
                    "metadata": {
                      "description": "Select the Log Analytics workspace from dropdown list",
                      "displayName": "Log Analytics workspace",
                      "strongType": "omsWorkspace"
                    },
                    "type": "String"
                  }
                },
                "PolicyDefinitionGroups": null,
                "PolicyDefinitions": [
                ...
                  {
                    "policyDefinitionReferenceId": "11507151035072608457",
                    "policyDefinitionId": "/providers/Microsoft.Management/managementGroups/ES/providers/Microsoft.Authorization/policyDefinitions/Deploy-Diagnostics-KeyVault",
                    "parameters": {
                      "logAnalytics": {
                        "value": "[parameters('logAnalytics')]"
                      }
                    }
                  }
                 ...
                ]
              },
              "ResourceId": "/providers/Microsoft.Management/managementGroups/ES/providers/Microsoft.Authorization/policySetDefinitions/Deploy-Diag-LogAnalytics",
              "ResourceName": "Deploy-Diag-LogAnalytics",
              "ResourceType": "Microsoft.Authorization/policySetDefinitions",
              "SubscriptionId": null
            },
```

## Deploy Key Vault

Because the Azure builtin Policy Deploy Diagnostic Settings for Key Vault to Log Analytics workspace already was part of an Enterprise-Scale reference architecture implementation, we can now test what happens if we deploy a Key Vault without the setting to stream the diagnostic settings to a central Log Analytics workspace

![](/assets//PolicyInitiative.png)

Use the following PowerShell code to deploy a Key Vault:

```PowerShell
<#
    Deploy Key Vault without Diagnostic Settings configured
#>

New-AzResourceGroup -Name 'es-demo-rg' -Location 'WestEurope'

New-AzKeyVault -Name 'es-demo-kv' -ResourceGroupName 'es-demo-rg' -Location 'WestEurope'
```

![Screenshot of PowerShell Key Vault Deployment](/assets//PowerShell-KeyVault-Deployment.png)

![Screenshot Key Vault Diagnostic Settings](/assets//KV-Diagnostic-Settings.png)

Let's wait until the Azure Policy remediation actions kicks-in.

![](/assets//KV-Diagnostic-Setting-via-Policy.png)

Please use the comments available below this blog post to share you insights regarding this Policy Driven Governance principle of Enterprise-Scale.

# References

[1] [Azure Architecture Blog - Enterprise-Scale and Azure Policy for policy-driven governance](https://techcommunity.microsoft.com/t5/azure-architecture-blog/enterprise-scale-and-azure-policy-for-policy-driven-governance/ba-p/1614060)

[2] [Microsoft Docs - What is Azure Policy](https://docs.microsoft.com/en-us/azure/governance/policy/overview)

[3] [Github - Azure Poliy Samples](https://github.com/Azure/azure-policy)

[4] [Microsoft Docs -  Get compliancy data of Azure resources](https://docs.microsoft.com/en-us/azure/governance/policy/how-to/get-compliance-data)

[5] [Microsoft Docs - Uderstand Azure Policy Effects](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/effects)

[6] [Microsoft Docs - How compliancy works](https://docs.microsoft.com/en-us/azure/governance/policy/how-to/get-compliance-data#how-compliance-works)

[7] [Microsoft Docs - Order of Evaluation](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/effects#order-of-evaluation)

[8] [Github Enterprise-Scale - AzOpsReference](https://github.com/Azure/Enterprise-Scale/tree/main/azopsreference)

[9] [Github Enterprise-Scale - Deploy Enterprise-Scale Reference implementation in your own environment ](https://github.com/Azure/Enterprise-Scale/blob/main/docs/EnterpriseScale-Deploy-reference-implentations.md)

[10] [Github Enterprise-Scale - Deploy Enterprise-Scale foundation for Wingtips](https://github.com/Azure/Enterprise-Scale/blob/main/docs/reference/wingtip/README.md)

[11] [Githb Enterprise-Scale - Introduction to Enterprise-Scale "in-a-box"](https://github.com/Azure/Enterprise-Scale/tree/main/docs/enterprise-scale-iab)

[12] [CloudSkew - Draw cloud architecture diagrams for free](https://www.cloudskew.com/)

[13] [Github Enterprise-Scale Getting started with infrastructure-as-code](https://github.com/Azure/Enterprise-Scale/blob/main/docs/Deploy/getting-started.md)

[14] [Github Action - AzOps](https://github.com/Azure/AzOps)

[15] [Microsoft Docs - What is an Azure landing zone](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone/)

[16] [Github Enterprise-Scale - Azure DevOps - Setup Guide](https://github.com/Azure/Enterprise-Scale/blob/main/docs/Deploy/setup-azuredevops.md)

[17] [Github AzOps Repository - Docker file](https://github.com/Azure/AzOps/blob/main/Dockerfile)

[18] [Docker image AzOps on Docker Hub](https://hub.docker.com/r/mscet/azops)

[19] [PlatformOps in a Microsoft Enterprise-scale landing zone](https://www.linkedin.com/pulse/platformops-microsoft-enterprise-scale-landing-zone-anders-bonde/)

[20] [AzGovViz - Azure Governance Visualizer](https://github.com/JulianHayward/Azure-MG-Sub-Governance-Reporting)

[21] [Deploy your own ARM templates with AzOps GitHub Actions](https://github.com/Azure/Enterprise-Scale/blob/main/docs/Deploy/deploy-new-arm.md)

[22] [Deploy Policy assignmen](https://github.com/Azure/Enterprise-Scale/blob/main/docs/Deploy/deploy-new-policy-assignment.md)

[23] [Enterprise-Scale Schema](https://github.com/Azure/Enterprise-Scale/blob/main/docs/Deploy/ES-schema.md)

[24] [AzAdvertizer](https://www.azadvertizer.net/info.html)

[25] [Wingtip Reference implemenation - What will be deployed](https://github.com/Azure/Enterprise-Scale/tree/main/docs/reference/wingtip#what-will-be-deployed)

[26] [Wingtip Azure Portal Custom deployment user interface](https://github.com/Azure/Enterprise-Scale/tree/main/docs/reference/wingtip/armTemplates)