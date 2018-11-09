---
layout: post
title: Parsing your ARM Template Output in VSTS Pipelines
categories: [Azure, ARM, VSTS, PowerShell]
tags: [Azure, ARM, VSTS, PowerShell]
comments: true
---
Did you know you can store your ARM template output in a VSTS variable and parse that data for later usage in your VSTS Release Pipeline?

If you are using the <a href="https://github.com/Microsoft/vsts-tasks/blob/master/Tasks/AzureResourceGroupDeploymentV2/README.md" target="_blank">Azure Resource Group Deployment Task</a> in your VSTS Release Definitions you now have the option to store the output values from your ARM template in the Deployments Output section.

![](/assets/armtemplateoutputfield.png)

In this blog post I explain how you can use the Deployments Output values in the rest of your Release Pipeline.

**ARM Template**

Before we can start, we first need to have an ARM Template which some output. In this example I don't deploy any Azure Resources I just want to output something.

armtemplate.json

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "firstName": {
      "type": "string",
      "metadata": {
        "description": "The first name of user"
      }
    },
      "lastName": {
        "type": "string",
        "metadata": {
          "description": "The last name of user"
        }
      }
    },
  "variables": {},
  "resources": [],
  "outputs": {
    "firstNameOutput": {
      "value": "[parameters('firstName')]",
      "type": "string"
    },
    "lastNameOutput": {
      "value": "[parameters('lastName')]",
      "type": "string"
    }
  }
}
```

This ARM Deployment has 2 parameters for firstName and lastName.

armtemplate.parameters.json

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "firstName": {
      "value": "Stefan"
    },
    "lastName": {
      "value": "Stranger"
    }
  }
}
```
When you deploy above ARM Template you would see the following output being shown in the Azure Portal.

![](/assets/armdeploymentoutput.png)

**PowerShell script**

For parsing the output of the ARM Template we are going to create a PowerShell script.

Folder view in Visual Studio:
![](/assets/folderviewinvs.png)

Parse-ARMOutput.ps1:

```powershell
param (
    [Parameter(Mandatory=$true)][string]$ARMOutput
    )

#region Convert from json
$json = $ARMOutput | convertfrom-json
#endregion

#region Parse ARM Template Output
Write-Output -InputObject ('Hello {0} {1}' -f $json.firstNameOutput.value, $json.lastNameOutput.value)
#endregion
```

**Build and Release Definition**

After committing the ARM Template and script files to your Repository it's time to create a Build and Release Definition in Visual Studio Team Services (VSTS).

Build Definition:
![](/assets/builddefinition.png)

Release Definition:
![](/assets/releasedefoutput.png)

Make sure you define Deployment outputs variable in the Azure Resource Group Deployment VSTS Task. This value is going to be used in the next step of the Release Pipeline.

Create a new Release Task for parsing the output from the Deployment Outputs section using the PowerShell script created earlier.

**PowerShell VSTS Task**

For parsing the Deployment Outputs from the Azure Resource Group Task we are using the PowerShell Task from VSTS.

![](/assets/parsingtask.png)

When you now trigger a release you see that the output from the ARM Template deployment is being parsed in the PowerShell Script VSTS task.

![](/assets/PoshScriptTask.png)

Hope you find some more scenario's in which you can use the Deployment Outputs option from the Azure Resource Group Task.



**References:**

* <a href="https://github.com/Microsoft/vsts-tasks/blob/master/Tasks/AzureResourceGroupDeploymentV2/README.md" target="_blank">Azure Resource Group Deployment Task</a> 
* <a href="https://marketplace.visualstudio.com/items?itemName=keesschollaart.arm-outputs" target="_blank">ARM Outputs VSTS Extension by Kees Schollaart</a>