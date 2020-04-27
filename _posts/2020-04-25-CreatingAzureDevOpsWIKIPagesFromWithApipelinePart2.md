---
layout: post
title: Creating Azure DevOps WIKI Pages from within a pipeline - part 2
categories: [CI/CD, WIKI]
tags: [CI/CD, WIKI, Markdown]
comments: true
---

**Contents**

- [Scenario](#scenario)
- [Solution](#solution)
  - [Documentation details](#documentation-details)
  - [Metadata file](#metadata-file)
  - [ARM Template](#arm-template)
  - [ARM Preview](#arm-preview)
    - [Mermaid](#mermaid)
  - [PSDocs Template](#psdocs-template)
  - [Create Markdown deployment document and publishing script](#create-markdown-deployment-document-and-publishing-script)
  - [Azure DevOps Pipeline](#azure-devops-pipeline)
    - [Permissions on Azure DevOps wiki repository](#permissions-on-azure-devops-wiki-repository)
    - [Yaml pipeline](#yaml-pipeline)
- [Screenshots from all the components](#screenshots-from-all-the-components)
  - [Repository](#repository)
  - [Yaml Pipeline Run](#yaml-pipeline-run)
  - [Azure Resource Group and resources](#azure-resource-group-and-resources)
  - [Wiki Markdown documentation](#wiki-markdown-documentation)

This is part 2 of a blog post series about creating Azure DevOps WIKI pages from within a Pipeline.

If you have not read [part 1](https://stefanstranger.github.io/2020/04/12/CreatingAzureDevOpsWIKIPagesFromWithApipeline/), please do so first.

# Scenario

The scenario for demonstrating the creation of an Azure DevOps WIKI pages from within a Pipeline is the creation of documentation (WIKI page) for each Azure Resource Deployment being made via an Azure DevOps Pipeline.

Below a screenshot of the end result, being documentation for an Azure Resource Deployment of an Azure Logic App.

![Screenshot WIKI documentation ARM deployment](/assets/2020-04-25-01.png)

The goal is to have a similar Wiki document created for each of the Azure Resource Deployments being made via an Azure DevOps Pipeline.

# Solution

## Documentation details

The documentation contains the following sections:
* Title
* Parameters
* Variables
* Resources
* Deployment information
* ARM Preview

The following table shows the sources for each of above sections:

| Section | Source |
|----------|----------|
| Title | Metadata file |
| Parameters | ARM Template file |
| Variables | ARM Template file |
| Resources | ARM Template file |
| Deployment information | PSDocs template file |
| ARM Preview | ARM Template file |

## Metadata file

This metadata file is something I re-used from the documentation of the PSDocs PowerShell Module. 

## ARM Template

For the deployment I used an ARM Template file to deploy an Azure Logic App which I used in a customer demo about Automation some time ago.

**Note:**
Not showing the parameter file because that was not used for the creation of documentation.

template.json
```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "connections_office365_name": {
            "defaultValue": "office365",
            "type": "String"
        },
        "workflows_logicapp_demo_name": {
            "defaultValue": "logicapp-demo",
            "type": "String"
        },
        "connections_visualstudioteamservices_name": {
            "defaultValue": "visualstudioteamservices",
            "type": "String"
        }
    },
    "variables": {

        "storageName": "[concat(toLower(parameters('storageNamePrefix')), uniqueString(resourceGroup().id))]"

    },
    "resources": [
        {
            "type": "Microsoft.Web/connections",
            "apiVersion": "2016-06-01",
            "name": "[parameters('connections_office365_name')]",
            "location": "westeurope",
            "properties": {
                "displayName": "john.doe@contoso.com",
                "customParameterValues": {
                },
                "api": {
                    "id": "[concat('/subscriptions/91eebaf8-59ae-4b4e-88aa-36ce41d7fb2d/providers/Microsoft.Web/locations/westeurope/managedApis/', parameters('connections_office365_name'))]"
                }
            }
        },
        {
            "type": "Microsoft.Web/connections",
            "apiVersion": "2016-06-01",
            "name": "[parameters('connections_visualstudioteamservices_name')]",
            "location": "westeurope",
            "properties": {
                "displayName": "john.doe@contoso.com",
                "customParameterValues": {
                },
                "api": {
                    "id": "[concat('/subscriptions/91eebaf8-59ae-4b4e-88aa-36ce41d7fb2d/providers/Microsoft.Web/locations/westeurope/managedApis/', parameters('connections_visualstudioteamservices_name'))]"
                }
            }
        },
        {
            "type": "Microsoft.Logic/workflows",
            "apiVersion": "2017-07-01",
            "name": "[parameters('workflows_logicapp_demo_name')]",
            "location": "westeurope",
            "dependsOn": [
                "[resourceId('Microsoft.Web/connections', parameters('connections_office365_name'))]",
                "[resourceId('Microsoft.Web/connections', parameters('connections_visualstudioteamservices_name'))]"
            ],
            "properties": {
                "state": "Enabled",
                "definition": {
                    "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {
                        "$connections": {
                            "defaultValue": {
                            },
                            "type": "Object"
                        }
                    },
                    "triggers": {
                        "When_code_is_pushed_(Git)": {
                            "recurrence": {
                                "frequency": "Minute",
                                "interval": 3
                            },
                            "splitOn": "@triggerBody()?['value']",
                            "type": "ApiConnection",
                            "inputs": {
                                "host": {
                                    "connection": {
                                        "name": "@parameters('$connections')['visualstudioteamservices']['connectionId']"
                                    }
                                },
                                "method": "get",
                                "path": "/gitpushed_trigger/@{encodeURIComponent('Automation-Demo-Project')}/_apis/git/repositories/@{encodeURIComponent('Automation-Demo-Project')}/pushes",
                                "queries": {
                                    "account": "johndoe"
                                }
                            }
                        }
                    },
                    "actions": {
                        "Send_an_email_(V2)": {
                            "runAfter": {
                            },
                            "type": "ApiConnection",
                            "inputs": {
                                "body": {
                                    "Body": "<p>New code is pushed by @{triggerBody()?['pushedBy']?['displayName']} from repository @{triggerBody()?['repository']?['name']}.<br>\n<br>\nOpen the following url for more info:  @{triggerBody()?['repository']?['remoteUrl']}.</p>",
                                    "Subject": "Demo New code is pushed by @{triggerBody()?['pushedBy']?['displayName']}",
                                    "To": "john.doe@contoso.com"
                                },
                                "host": {
                                    "connection": {
                                        "name": "@parameters('$connections')['office365']['connectionId']"
                                    }
                                },
                                "method": "post",
                                "path": "/v2/Mail"
                            }
                        }
                    },
                    "outputs": {
                    }
                },
                "parameters": {
                    "$connections": {
                        "value": {
                            "office365": {
                                "connectionId": "[resourceId('Microsoft.Web/connections', parameters('connections_office365_name'))]",
                                "connectionName": "office365",
                                "id": "/subscriptions/91eebaf8-59ae-4b4e-88aa-36ce41d7fb2d/providers/Microsoft.Web/locations/westeurope/managedApis/office365"
                            },
                            "visualstudioteamservices": {
                                "connectionId": "[resourceId('Microsoft.Web/connections', parameters('connections_visualstudioteamservices_name'))]",
                                "connectionName": "visualstudioteamservices",
                                "id": "/subscriptions/91eebaf8-59ae-4b4e-88aa-36ce41d7fb2d/providers/Microsoft.Web/locations/westeurope/managedApis/visualstudioteamservices"
                            }
                        }
                    }
                }
            }
        }
    ]
}
```
The ARM Template was iterated with the following code to retrieve the information I was looking for.

```PowerShell
Function Get-ARMTemplate {
    param (
        [Parameter(Mandatory = $True)]
        [String]$ARMTemplatePath
    )

    $template = Get-Content $ARMTemplatePath | ConvertFrom-Json

    #Parameters
    $Parameters = foreach ($property in $template.parameters.PSObject.Properties) {
        [PSCustomObject]@{
            Name        = $property.Name
            Description = $property.Value.metadata.description
        }
    }

    #Variables
    $Variables = foreach ($property in $template.variables.PSObject.Properties) {
        [PSCustomObject]@{
            Name  = $property.Name
            Value = $property.Value
        }
    }


    #Resources
    $Resources = foreach ($property in $template.resources) {
        [PSCustomObject]@{
            Name      = $property.Name
            Type      = $property.Type
            Location  = $property.Location
            DependsOn = $property.DependsOn
        }
    }


    return [PSCustomObject]@{
        'Parameters' = $Parameters
        'Variables'  = $Variables
        'Resources'  = $Resources
    }
}
```
## ARM Preview

For the ARM Preview part of the document I used [Mermaid](https://mermaid-js.github.io/mermaid/#/) to create an diagram overview of the Resource Dependencies.

If you look at the ARM Template file above you see that Azure Resource Type Microsoft.Logic/Workflows has a dependency on Microsoft.Web/connections resources.

![Mermaid Graph](/assets/2020-04-25-03.png)


![ARM Template Resource dependencies](/assets/2020-04-25-02.png)

### Mermaid
 Azur DevOps Wiki supports [Mermaid Diagrams](https://docs.microsoft.com/en-us/azure/devops/project/wiki/wiki-markdown-guidance?view=azure-devops#add-mermaid-diagrams-to-a-wiki-page).

Mermaid is a simple markdown-like script language for generating charts from text via JavaScript.

With some custom PowerShell code and the PowerShell Mermaid Module PSMermaid a very simple ARM Preview Diagram was created. 

You can install the PowerShell Module with the following code:

```PowerShell
Install-Module -Name PSMermaid -Scope CurrentUser
```

For the creation of the ARM Preview Diagram I used the following PowerShell Functions. First we use the earlier shared Helper Function to parse the ARM Template file. The next helper Function New-MermaildResourceDiagram creates the Mermaid diagram code to be used in the Markdown document.

```PowerShell
Function Get-ARMTemplate {
    param (
        [Parameter(Mandatory = $True)]
        [String]$ARMTemplate
    )

    $template = Get-Content $ARMTemplate | ConvertFrom-Json

    #Parameters
    $Parameters = foreach ($property in $template.parameters.PSObject.Properties) {
        [PSCustomObject]@{
            Name        = $property.Name
            Description = $property.Value.metadata.description
        }
    }

    #Variables
    $Variables = foreach ($property in $template.variables.PSObject.Properties) {
        [PSCustomObject]@{
            Name  = $property.Name
            Value = $property.Value
        }
    }


    #Resources
    $Resources = foreach ($property in $template.resources) {
        [PSCustomObject]@{
            Name      = $property.Name
            Type      = $property.Type
            Location  = $property.Location
            DependsOn = $property.DependsOn
        }
    }


    return [PSCustomObject]@{
        'Parameters' = $Parameters
        'Variables'  = $Variables
        'Resources'  = $Resources
    }
}

# Function to create a simple ARM Preview Diagram
Function New-MermaidResourceDiagram {

    [CmdLetBinding()]
    Param ([Parameter (Mandatory = $true)]
        [String] $ARMTemplate
    )

    $InputObject = [pscustomobject]@{
        'ARMTemplate' = Get-ARMTemplate -ARMTemplate $ARMTemplate
    }

    $i = 0

@'
:::mermaid
graph TD
'@
    Foreach ($Resource in $($InputObject.ARMTemplate.Resources)) {
        if ($Resource.DependsOn) {
            m-node -Id $Resource.Type -Attributes @{
                LinkTo = Foreach ($Depends in $Resource.DependsOn) {
                    #Refactor ARM template depends on string
                    ('{0}[{1}]' -f $i, ([regex]::matches($Depends, "'(Microsoft.*?)'").Value).Trim("'", " ").trim("/", " "))
                    $i++
                }
            }
        }
    }
@'
:::
'@
}
```

When you run the Function New-MermaidResourceDiagram you get the following output:

```md
:::mermaid
graph TD
Microsoft.Logic/workflows-->0[Microsoft.Web/connections]
Microsoft.Logic/workflows-->1[Microsoft.Web/connections]
:::
```

## PSDocs Template

To create the Markdown document the following PSDocs Template is being used.

arm-template.doc.ps1
```PowerShell
<#
    PowerShell PSDocs Document to create an ARM Template Markdown Template.

    Requirements:
    - PSDocs PowerShell Module (Install-Module -Name PSDocs)

#>

# Description: A definition to generate markdown for an ARM template
document 'arm-template' {

    # Table of Contents
    '[[_TOC_]]'

    # Set document title
    Title $InputObject.metadata.itemDisplayName

    # Write opening line
    $InputObject.metadata.Description

    # Add each parameter to a table
    Section 'Parameters' {
        $InputObject.ARMTemplate.parameters | Table -Property @{ Name = 'Parameter name'; Expression = { $_.Name } }, Description
    }

    Section 'Variables' {
        $InputObject.ARMTemplate.Variables | Table -Property @{ Name = 'Variable name'; Expression = { $_.Name } }, Value
    }

    Section 'Resources' {
        $InputObject.ARMTemplate.Resources | Table -Property @{ Name = 'Resource name'; Expression = { $_.Name } }, Type, Location
    }

    # Generate example command line
    Section 'Use the template' {
        Section 'PowerShell' {
            'New-AzResourceGroupDeployment -Name <deployment-name> -ResourceGroupName <resource-group-name> -TemplateFile <path-to-template>' | Code powershell
        }

        Section 'Azure CLI' {
            'az group deployment create --name <deployment-name> --resource-group <resource-group-name> --template-file <path-to-template>' | Code text
        }
    }

    Section 'ARM Preview' {
        $InputObject.ARMPreview
    }
}
```
## Create Markdown deployment document and publishing script

If we put everything together in one script to create the ARM Template Markdown document and publish the document to the Azure DevOps wiki the following code can be used.

Create-DeploymentDocument.ps1
```PowerShell
<#
    PowerShell script to create a Markdown Azure deployment document and store document in Azure DevOps Wiki.

    Requirements:
    - PowerShell Modules
      - PSDocs
      - PSMermaid
    
    - Azure DevOps Permissions
      - Project Collection Build Service on Azure DevOps Wiki Repository needs Contribute Allow.
#>

[CmdLetBinding()]
Param (
    [Parameter (Mandatory = $True)]
    [String] $ARMTemplate,
    [Parameter (Mandatory = $True)]
    [String] $MetaData,
    [Parameter (Mandatory = $True)]
    [String] $MarkDownTemplate,
    [Parameter (Mandatory = $True)]
    [String] $WikiPageName
)

#region variables
$OrganizationName = 'stefanstranger'
#endregion

#region Helper Function to parse content from ARM Template
Function Get-ARMTemplate {
    param (
        [Parameter(Mandatory = $True)]
        [String]$ARMTemplate
    )

    $template = Get-Content $ARMTemplate | ConvertFrom-Json

    #Parameters
    $Parameters = foreach ($property in $template.parameters.PSObject.Properties) {
        [PSCustomObject]@{
            Name        = $property.Name
            Description = $property.Value.metadata.description
        }
    }

    #Variables
    $Variables = foreach ($property in $template.variables.PSObject.Properties) {
        [PSCustomObject]@{
            Name  = $property.Name
            Value = $property.Value
        }
    }


    #Resources
    $Resources = foreach ($property in $template.resources) {
        [PSCustomObject]@{
            Name      = $property.Name
            Type      = $property.Type
            Location  = $property.Location
            DependsOn = $property.DependsOn
        }
    }


    return [PSCustomObject]@{
        'Parameters' = $Parameters
        'Variables'  = $Variables
        'Resources'  = $Resources
    }
}
#endregion

#region Helper function to get Meta data for ARM Template deployment
Function Get-TemplateMetadata {
    param (
        [Parameter(Mandatory = $True)]
        [String]$MetaData
    )

    process {
        $metadata = Get-Content $MetaData | ConvertFrom-Json;
        return $metadata;
    }
}
#endregion

#region Helper Function to create a simple ARM Preview Diagram
Function New-MermaidResourceDiagram {

    [CmdLetBinding()]
    Param ([Parameter (Mandatory = $true)]
        [String] $ARMTemplate
    )

    $InputObject = [pscustomobject]@{
        'ARMTemplate' = Get-ARMTemplate -ARMTemplate $ARMTemplate
    }

    $i = 0

@'
:::mermaid
graph TD
'@
    Foreach ($Resource in $($InputObject.ARMTemplate.Resources)) {
        if ($Resource.DependsOn) {
            m-node -Id $Resource.Type -Attributes @{
                LinkTo = Foreach ($Depends in $Resource.DependsOn) {
                    #Refactor ARM template depends on string
                    ('{0}[{1}]' -f $i, ([regex]::matches($Depends, "'(Microsoft.*?)'").Value).Trim("'", " ").trim("/", " "))
                    $i++
                }
            }
        }
    }
@'
:::
'@
}
#endregion

#region Code to create Markdown Deployment document using PSDocs Module
$InputObject = [pscustomobject]@{
    'MetaData'    = Get-TemplateMetaData -MetaData $MetaData
    'ARMTemplate' = Get-ARMTemplate -ARMTemplate $ARMTemplate
    'ARMPreview' = New-MermaidResourceDiagram -ARMTemplate $ARMTemplate
}
# Configure MarkDown options
$options = New-PSDocumentOption -Option @{ 'Markdown.UseEdgePipes' = 'Always'; 'Markdown.ColumnPadding' = 'None' };

$Markdown = Invoke-PSDocument -Path ('{0}' -f $MarkDownTemplate) -InputObject $InputObject -Option $options -PassThru
#endregion


#region to Publish Markdown Deployment document to Azure DevOps Wiki

#region dynamic variables
$ProjectName = $($env:SYSTEM_TEAMPROJECT) #'Automation-Demo-Project
$WikiName = ('{0}.wiki' -f $($env:SYSTEM_TEAMPROJECT)) #'Automation-Demo-Project.wiki'
#endregion

#region Create WIKI page
$uri = ('https://dev.azure.com/{0}/{1}/_apis/wiki/wikis/{2}/pages?path={3}&api-version=5.0' -f $OrganizationName, $ProjectName, $WikiName, $WikipageName)

$Header = @{
    'Authorization' = ('Bearer {0}' -f $($env:SYSTEM_ACCESSTOKEN)) 
}

$params = @{
    'Uri'         = $uri
    'Headers'     = $Header
    'Method'      = 'Put'
    'ContentType' = 'application/json; charset=utf-8'
    'body'        = @{content = $Markdown; } | ConvertTo-Json
}

$params | ConvertTo-Json -Compress -Depth 5

Invoke-RestMethod @params
#endregion

#endregion
```

## Azure DevOps Pipeline

In the final step we are going to put everything together in an Azure DevOps Pipeline. The following steps need to be executed.

1. Configure permissions on Azure DevOps wiki repository to publish document
1. Deploy the Azure Logic App
1. Create Markdown documentation of Azure Resource deployment and publish to wiki


### Permissions on Azure DevOps wiki repository

To have the deployment Markdown document being published to the Azure DevOps wiki (repository) the security on the Wiki needs to be configured. Confgure the Contribute setting to allow for the Project Collection Build Service.

![wiki Security Settings](/assets/2020-04-25-04.png)

![Contribute Allow](/assets/2020-04-25-05.png)


### Yaml pipeline

The yaml pipeline to execute above steps should look like this:

```yaml
# Build Logic App Demo Pipeline

name: DeployLogicApp_$(Date:yyyyMMdd)$(Rev:.r)

trigger:
  branches:
    include:
      - master
  paths:
    include:
      - /LogicApp/Templates/*

# Set variables
variables:
  ResourceGroupName: "logicapp-demo-rg"
  azureSubscription: "[enter Azure Subscription Service Connection Name]"
  location: "West Europe"
  vmImageName: "windows-latest"

stages:
  - stage: deploy
    displayName: Deploy stage
    
        displayName: deploy
        pool:
          vmImage: $(vmImageName)
        steps:
          - task: AzureCLI@2
            displayName: "Azure CLI"
            inputs:
              azureSubscription: $(azureSubscription)
              scriptType: pscore
              scriptLocation: inlineScript
              inlineScript: |
                "az group create --location $(location) --name $(ResourceGroupName)"          
          - task: PowerShell@2
            name: create_deploymentname
            displayName: 'Create Deployment Name variable'            
            enabled: true
            inputs:
              targettype: 'inline'
              script: |
                $DeploymentName = ('deployment-{0}') -f $(Get-Date).ToString('yyyyMMddHHmmss')
                Write-Host ('Creating variable DeploymentName with value {0}' -f $DeploymentName)
                Write-Host "##vso[task.setvariable variable=DeploymentName;isOutput=true]$DeploymentName"
          - task: AzureResourceGroupDeployment@2
            displayName: "Azure Deployment:Create Or Update Resource Group action on $(ResourceGroupName)"
            inputs:
              azureSubscription: $(azureSubscription)
              resourceGroupName: $(ResourceGroupName)
              location: "$(location)"
              templateLocation: "Linked artifact"
              csmFile: "LogicApp/Templates/template.json"
              csmParametersFile: "LogicApp/Templates/parameters.json"
              deploymentMode: "Incremental"
              deploymentName: $(DeploymentName)
      - job: documentation
        displayName: documentation
        dependsOn: deploy
        variables:
          DeploymentName: $[ dependencies.deploy.outputs['create_deploymentname.DeploymentName'] ]
        pool:
          vmImage: $(vmImageName)
        steps:
          - checkout: self
            persistCredentials: true
          - task: PowerShell@2
            name: check_deploymentName
            displayName: 'Check for DeploymentName'            
            enabled: true
            inputs:
              targettype: 'inline'
              script: |
                Write-Host ('Value of DeploymentName variable is: {0}' -f $($env:DeploymentName))
          - task: PowerShell@2
            name: install_modules
            displayName: 'Install PSDocs and PSMermaid PowerShell Modules'            
            enabled: true
            inputs:
              targettype: 'inline'
              script: |
                $null = Install-PackageProvider -Name NuGet -Force -Scope CurrentUser
                Install-Module -Name PSDocs -Scope CurrentUser -Force
                Install-Module -Name PSMermaid -Scope CurrentUser -Force
          - task: PowerShell@2
            name: create_arm_document
            displayName: 'Create ARM Template Documentation'
            inputs:
              filePath: '$(System.DefaultWorkingDirectory)\LogicApp\Markdown\Create-DeploymentDocument.ps1'
              arguments: '-ARMTemplate $(System.DefaultWorkingDirectory)\LogicApp\Templates\template.json -MetaData $(System.DefaultWorkingDirectory)\LogicApp\Markdown\metadata.json -MarkdownTemplate $(System.DefaultWorkingDirectory)\LogicApp\Markdown\arm-template.doc.ps1 -WikiPageName $(DeploymentName)'
              errorActionPreference: 'Stop'
            env:
              system_accesstoken: $(System.AccessToken)
              system_teamproject: $(System.TeamProject)
```

# Screenshots from all the components

## Repository

![Screenshot repo](/assets/2020-04-25-06.png)

## Yaml Pipeline Run

![Screenshot yaml pipeline run](/assets/2020-04-25-07.png)

## Azure Resource Group and resources

![Screenshot Azure Resource Group](/assets/2020-04-25-08.png)

## Wiki Markdown documentation

![Markdown document](/assets/2020-04-25-01.png)


**References:**

* [Creating Azure DevOps WIKI Pages from within a pipeline - part 1](https://stefanstranger.github.io/2020/04/12/CreatingAzureDevOpsWIKIPagesFromWithApipeline/)
* [PSDocs ARM Template example](https://github.com/BernieWhite/PSDocs/blob/master/docs/scenarios/arm-template/arm-template.md)
* [Blog Post - Dynamically create README Files from Azure DevOps Pipeline and Commit to Repository](https://mscloud.be/azure/Dynamically-create-README-from-azdo-pipeline-and-commit-repository/)
* [Azure DevOps WIKI support for Mermaid diagrams](https://docs.microsoft.com/en-us/azure/devops/project/wiki/wiki-markdown-guidance?view=azure-devops#add-mermaid-diagrams-to-a-wiki-page)
* [Mermaid](https://mermaid-js.github.io/mermaid/#/README)
* [PowerShell PSMermaid Module](https://github.com/pugbg/psmodules/tree/master/modules/PSMermaid)
* [Passing variables from stage to stage in Azure DevOps Release Pipelines](https://stefanstranger.github.io/2019/06/26/PassingVariablesfromStagetoStage/)
* [How to pass variables in Azure Pipelines YAML tasks](https://medium.com/microsoftazure/how-to-pass-variables-in-azure-pipelines-yaml-tasks-5c81c5d31763)

