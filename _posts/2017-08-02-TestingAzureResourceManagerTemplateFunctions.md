---
layout: post
title: Testing Azure Resource Manager template functions
categories: [Azure, PowerShell]
tags: [Azure, PowerShell]
---
In Azure Resource Manager (ARM) you can use **template functions** to help deploy resources in Azure.

I often find it difficult to develop ARM Templates with template functions without the option to debug while developing the ARM templates.

For an overview of the Azure Resource Manager templates go <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-template-functions" target="_blank">here</a>.

**ARM Template Output**
I just found out you could use the ARM template outputs.

In the Outputs section, you specify values that are returned from a deployment. For example, you could return the URI to access a deployed resource.

Example:
```json
"outputs": {
    "<outputName>" : {
        "type" : "<type-of-output-value>",
        "value": "<output-value-expression>"
    }
}
```

You can use the **outputs section** to deploy nothing but output your Template Functions output.

**Scenario:**

I want to test the output for the <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-template-functions-logical#if" target="_blank">If Condition</a>. Let's check if the parameter input for FirstName is "Stefan" or someone else.

We are going to use PowerShell with the Azure RM Modules to test Azure Template Functions.

```powershell
<#
    Testing Azure ARM Functions
    https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-template-functions
#>

#region Variables
$ResourceGroupName = 'armfunctions-rg'
$Location = 'WestEurope'
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

#region create Resource Group to test Azure Template Functions
If (!(Get-AzureRMResourceGroup -name $ResourceGroupName -Location $Location -ErrorAction SilentlyContinue)) {
    New-AzureRmResourceGroup -Name $ResourceGroupName -Location $Location
}

#endregion

# region Example for if condition
$template = @'
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "firstName": {
      "type": "string",
      "metadata": {
        "description": "The First Name of User"
      }
    }
     },
    "variables": { },
    "resources": [ ],
    "outputs": {
        "ifOutput": {
            "value": "[if(equals(parameters('firstName'),'Stefan'), 'You are Stefan', 'You are not Stefan')]",
            "type": "string"
        }
    }
}
'@
#endregion

$template | Out-File -File c:\temp\template.json -Force

#region Test ARM Template
Test-AzureRmResourceGroupDeployment -ResourceGroupName $ResourceGroupName -TemplateFile c:\temp\template.json -OutVariable testarmtemplate
#endregion

#region Deploy ARM Template with local Parameter file
$result = (New-AzureRmResourceGroupDeployment -ResourceGroupName $ResourceGroupName -TemplateFile c:\temp\template.json)
$result
#endregion
```
After first authenticating to Azure we create a resource group if not already created.

Next step is verifying if the ARM Template content is correct using the Test-AzureRmResourceGroupDeployment cmdlet.

![](/assets/testarmtemplate.png)

All seems to be fine, let's continue with the deployment in Azure. Keep in mind we are not deploying any Azure Resources but only want the outputs returned.

When configuring the parameter value FirstName with the value "Stefan" we expect to see the string "You are Stefan" returned.

![](/assets/outputtrue.png)

Now we enter a different value as input for the FirstName parameter to verify what happens when the If condition is false.

![](/assets/outputfalse.png)

With this information you can use the Template functions in **the condition element** of your ARM Templates.

```json
{
    "condition": "[equals(parameters('newOrExisting'),'new')]",
    "type": "Microsoft.Storage/storageAccounts",
    "name": "[variables('storageAccountName')]",
    "apiVersion": "2017-06-01",
    "location": "[resourceGroup().location]",
    "sku": {
        "name": "[variables('storageAccountType')]"
    },
    "kind": "Storage",
    "properties": {}
}
```

Hope you found this interesting.

**References:**
* <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-template-functions" target="_blank">Azure Resource Manager template functions</a>
* <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates" target="_blank">Understand the structure and syntax of Azure Resource Manager templates</a>
