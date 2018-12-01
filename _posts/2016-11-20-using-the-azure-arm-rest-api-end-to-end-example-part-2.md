---
layout: post
title: Using the Azure ARM REST API – End to end Example Part 2
date: 2016-11-20 09:15
author: stefan stranger
comments: true
categories: [ARM, ARM, Automation, Automation, Azure, Azure, PowerShell]
---
<p>This blog post is a continuation of the <a href="https://blogs.technet.microsoft.com/stefan_stranger/2016/11/04/using-the-azure-arm-rest-api-end-to-end-example-part-1/" target="_blank">third blog post</a> in this Azure (ARM) REST API series. Please review the following blog posts first if you have not done yet:</p> <ol> <li><a href="https://blogs.technet.microsoft.com/stefan_stranger/2016/10/21/using-the-azure-arm-rest-apin-get-access-token/">Using the Azure ARM REST API – Get Access Token</a>  <li><a href="https://blogs.technet.microsoft.com/stefan_stranger/2016/10/29/using-the-azure-arm-rest-api/" target="_blank">Using the Azure ARM REST API – Get Subscription Information</a>  <li><a href="https://blogs.technet.microsoft.com/stefan_stranger/2016/11/04/using-the-azure-arm-rest-api-end-to-end-example-part-1/" target="_blank">Using the Azure ARM REST API – End to end Example Part 1</a></li></ol> <p>In this blog post we will create a Virtual Machine using an ARM template and call an Azure Automation Runbook to stop the Virtual Machine.</p> <p><strong>Create a Virtual Machine using an ARM Template</strong></p> <p>The easiest way to find ARM templates to create a Virtual Machine is via the <a href="https://github.com/Azure/azure-quickstart-templates" target="_blank">Azure Quickstart templates on Github</a>. This repository contains all currently available Azure Resource Manager templates contributed by the community. A searchable template index is maintained at <a href="https://azure.microsoft.com/en-us/documentation/templates/">https://azure.microsoft.com/en-us/documentation/templates/</a>.</p> <p>If we search the repository we can find a <a href="https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows" target="_blank">very simple simple deployment of an Windows VM</a>. This template allows you to deploy a simple Windows VM using a few different options for the Windows version, using the latest patched version. This will deploy a D1 size VM in the resource group location and return the fully qualified domain name of the VM. </p> <p>First step is using the <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-vm-simple-windows%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"> </a>option to test the deployment before we are going to use the Azure ARM REST API to deploy the VM. When you click on the <u><em>Deploy to Azure button</em></u> and login to your Azure Subscription you are presented with the needed parameters for this ARM Template.</p> <p><a href="https://msdnshared.blob.core.windows.net/media/2016/11/image460.png"><img title="image" style="border-left-width: 0px;border-right-width: 0px;border-bottom-width: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border-top-width: 0px" border="0" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb375.png" width="612" height="572"></a></p> <p>Configure the correct parameter values and check the successful deployment of the simple Windows VM and click on Purchase. Wait for the deployment to finish and check if this is the kind of VM you want to deploy in Azure. </p> <p>You would see something like this if everything went ok.</p> <p><a href="https://msdnshared.blob.core.windows.net/media/2016/11/image461.png"><img title="image" style="border-left-width: 0px;border-right-width: 0px;border-bottom-width: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border-top-width: 0px" border="0" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb376.png" width="587" height="470"></a></p> <p>If you want you can also connect to the VM to do a final check on the deployment. When you are happy with the result you can delete the Resource Group with VM in it. </p> <p>We are going to download the <a href="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json" target="_blank">Azuredeploy.json</a> and <a href="https://github.com/Azure/azure-quickstart-templates/blob/master/101-vm-simple-windows/azuredeploy.parameters.json" target="_blank">azuredeploy.parameters.json</a> from the Github repository as input for our REST API call body.</p> <p>When we have the correct ARM Template for the Deployment the next step is use this ARM template according to the documentation on the <a href="https://msdn.microsoft.com/en-us/library/azure/dn790549.aspx" target="_blank">Azure Reference website on how to deploy ARM Templates using the REST API</a>.</p> <p>According to the documentation you need the use the following information for the web request:</p> <table cellspacing="0" cellpadding="2" width="1131" border="0"> <tbody> <tr> <td valign="top" width="81"><strong>Method</strong></td> <td valign="top" width="1048"><strong>Request URI</strong></td></tr> <tr> <td valign="top" width="81">PUT</td> <td valign="top" width="1048"> <p>https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/microsoft.resources/deployments/{deployment-name}?api-version={api-version}</p></td></tr></tbody></table> <p>&nbsp;</p> <p>The request body will have the <strong>ARM Template JSON with the parameter values</strong>. You would normally enter the parameter values manually or store them in the azuredeploy.parameters.json file but now we are adding the parameter values within the request body of the web request.</p> <p><a href="https://msdnshared.blob.core.windows.net/media/2016/11/image462.png"><img title="image" style="border-left-width: 0px;border-right-width: 0px;border-bottom-width: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border-top-width: 0px" border="0" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb377.png" width="885" height="316"></a></p> <p>Open both downloaded files in <a href="https://code.visualstudio.com" target="_blank">Visual Studio Code</a> and combine both files into the following web request body:</p><pre class="&rdquo;brush:"> {
    "properties": {
        "template": {
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {
                "adminUsername": {
                    "type": "string",
                    "metadata": {
                        "description": "Username for the Virtual Machine."
                    }
                },
                "adminPassword": {
                    "type": "securestring",
                    "metadata": {
                        "description": "Password for the Virtual Machine."
                    }
                },
                "dnsLabelPrefix": {
                    "type": "string",
                    "metadata": {
                        "description": "Unique DNS Name for the Public IP used to access the Virtual Machine."
                    }
                },
                "windowsOSVersion": {
                    "type": "string",
                    "defaultValue": "2016-Datacenter",
                    "allowedValues": [
                        "2008-R2-SP1",
                        "2012-Datacenter",
                        "2012-R2-Datacenter",
                        "2016-Nano-Server",
                        "2016-Datacenter-with-Containers",
                        "2016-Datacenter"
                    ],
                    "metadata": {
                        "description": "The Windows version for the VM. This will pick a fully patched image of this given Windows version."
                    }
                }
            },
            "variables": {
                "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'sawinvm')]",
                "nicName": "myVMNic",
                "addressPrefix": "10.0.0.0/16",
                "subnetName": "Subnet",
                "subnetPrefix": "10.0.0.0/24",
                "publicIPAddressName": "myPublicIP",
                "vmName": "SimpleWindowsVM",
                "virtualNetworkName": "MyVNET",
                "subnetRef": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('virtualNetworkName'), variables('subnetName'))]"
            },
            "resources": [
                {
                    "type": "Microsoft.Storage/storageAccounts",
                    "name": "[variables('storageAccountName')]",
                    "apiVersion": "2016-01-01",
                    "location": "[resourceGroup().location]",
                    "sku": {
                        "name": "Standard_LRS"
                    },
                    "kind": "Storage",
                    "properties": {}
                },
                {
                    "apiVersion": "2016-03-30",
                    "type": "Microsoft.Network/publicIPAddresses",
                    "name": "[variables('publicIPAddressName')]",
                    "location": "[resourceGroup().location]",
                    "properties": {
                        "publicIPAllocationMethod": "Dynamic",
                        "dnsSettings": {
                            "domainNameLabel": "[parameters('dnsLabelPrefix')]"
                        }
                    }
                },
                {
                    "apiVersion": "2016-03-30",
                    "type": "Microsoft.Network/virtualNetworks",
                    "name": "[variables('virtualNetworkName')]",
                    "location": "[resourceGroup().location]",
                    "properties": {
                        "addressSpace": {
                            "addressPrefixes": [
                                "[variables('addressPrefix')]"
                            ]
                        },
                        "subnets": [
                            {
                                "name": "[variables('subnetName')]",
                                "properties": {
                                    "addressPrefix": "[variables('subnetPrefix')]"
                                }
                            }
                        ]
                    }
                },
                {
                    "apiVersion": "2016-03-30",
                    "type": "Microsoft.Network/networkInterfaces",
                    "name": "[variables('nicName')]",
                    "location": "[resourceGroup().location]",
                    "dependsOn": [
                        "[resourceId('Microsoft.Network/publicIPAddresses/', variables('publicIPAddressName'))]",
                        "[resourceId('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]"
                    ],
                    "properties": {
                        "ipConfigurations": [
                            {
                                "name": "ipconfig1",
                                "properties": {
                                    "privateIPAllocationMethod": "Dynamic",
                                    "publicIPAddress": {
                                        "id": "[resourceId('Microsoft.Network/publicIPAddresses',variables('publicIPAddressName'))]"
                                    },
                                    "subnet": {
                                        "id": "[variables('subnetRef')]"
                                    }
                                }
                            }
                        ]
                    }
                },
                {
                    "apiVersion": "2015-06-15",
                    "type": "Microsoft.Compute/virtualMachines",
                    "name": "[variables('vmName')]",
                    "location": "[resourceGroup().location]",
                    "dependsOn": [
                        "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
                        "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
                    ],
                    "properties": {
                        "hardwareProfile": {
                            "vmSize": "Standard_D1"
                        },
                        "osProfile": {
                            "computerName": "[variables('vmName')]",
                            "adminUsername": "[parameters('adminUsername')]",
                            "adminPassword": "[parameters('adminPassword')]"
                        },
                        "storageProfile": {
                            "imageReference": {
                                "publisher": "MicrosoftWindowsServer",
                                "offer": "WindowsServer",
                                "sku": "[parameters('windowsOSVersion')]",
                                "version": "latest"
                            },
                            "osDisk": {
                                "name": "osdisk",
                                "vhd": {
                                    "uri": "[concat(reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))).primaryEndpoints.blob, 'vhds/osdisk.vhd')]"
                                },
                                "caching": "ReadWrite",
                                "createOption": "FromImage"
                            },
                            "dataDisks": [
                                {
                                    "name": "datadisk1",
                                    "diskSizeGB": "100",
                                    "lun": 0,
                                    "vhd": {
                                        "uri": "[concat(reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))).primaryEndpoints.blob, 'vhds/datadisk1.vhd')]"
                                    },
                                    "createOption": "Empty"
                                }
                            ]
                        },
                        "networkProfile": {
                            "networkInterfaces": [
                                {
                                    "id": "[resourceId('Microsoft.Network/networkInterfaces',variables('nicName'))]"
                                }
                            ]
                        },
                        "diagnosticsProfile": {
                            "bootDiagnostics": {
                                "enabled": "true",
                                "storageUri": "[reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))).primaryEndpoints.blob]"
                            }
                        }
                    }
                }
            ],
            "outputs": {
                "hostname": {
                    "type": "string",
                    "value": "[reference(variables('publicIPAddressName')).dnsSettings.fqdn]"
                }
            }
        },
        "mode": "Incremental",
        "parameters": {
            "adminUsername": {
                "value": "localadmin"
            },
            "adminPassword": {
                "value": "VerySecretPassword!"
            },
            "dnsLabelPrefix": {
                "value": "stsarestapidemovm"
            }
        }
    }
}
</pre>
<p>&nbsp;</p>
<p>Open <a href="http://httpmaster.net/" target="_blank">HttpMaster</a> or another tool you want to use for sending the web request and create the following request:</p><pre class="&rdquo;brush:">URL: {SubscriptionId}/resourcegroups/{ResourceGroupName}/providers/microsoft.resources/deployments/ARMDeployment?api-version=2016-09-01e
</pre>
<p><strong>* In HttpMaster we have stored the first part of the request (global url: <a title="https://management.azure.com/subscriptions" href="https://management.azure.com/subscriptions">https://management.azure.com/subscriptions</a>) If you are using a different tool make sure you use the complete endpoint.</strong></p>
<p><strong>Remark:</strong> </p>
<p>Check previous <a href="https://blogs.technet.microsoft.com/stefan_stranger/2016/11/04/using-the-azure-arm-rest-api-end-to-end-example-part-1/" target="_blank">blog post on how to use HttpMaster</a> and store parameter values in your HttpMaster project. </p>
<p><a href="https://msdnshared.blob.core.windows.net/media/2016/11/image463.png"><img title="image" style="border-left-width: 0px;border-right-width: 0px;border-bottom-width: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border-top-width: 0px" border="0" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb378.png" width="870" height="572"></a></p>
<p>When you execute the following requests from HttpMaster your simple VM should be deployed in Azure.</p>
<p><a href="https://msdnshared.blob.core.windows.net/media/2016/11/image464.png"><img title="image" style="border-left-width: 0px;border-right-width: 0px;border-bottom-width: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border-top-width: 0px" border="0" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb379.png" width="885" height="395"></a></p>
<p><strong>Pro tip:</strong></p>
<p>With the <a href="http://httpmaster.net/buy" target="_blank">professional edition of HttpMaster</a> you can use <strong><a href="http://httpmaster.net/features#chaining" target="_blank">chaining</a></strong> and <strong><a href="http://httpmaster.net/features#execution_groups" target="_blank">execution groups</a></strong> to streamline the requests even better compared to the free version.<br>When using the <strong>free version of HttpMaster</strong> make sure you store the AccessToken in the Header request. See the <a href="https://blogs.technet.microsoft.com/stefan_stranger/2016/11/04/using-the-azure-arm-rest-api-end-to-end-example-part-1/" target="_blank">Using the Azure ARM REST API – End to end Example Part 1</a> blog post for more information on how to store the Accesstoken. </p>
<p>If we now check in Azure, the simple VM should be deployed.</p>
<p>When the deployment is finished we can continue with calling the runbook to stop the VM we just created with the ARM Template.</p>
<p><strong>Stop a VM using a Runbook in Azure Automation</strong></p>
<p>If you have not created a Stop VM runbook in Azure Automation please do so before continuing. You can browse the Gallery for examples if you need some help.</p>
<p>I created the following PowerShell script Runbook which can be called from a webhook. More info on creating webhooks can be found <a href="https://docs.microsoft.com/en-us/azure/automation/automation-webhooks" target="_blank">here</a>.</p><pre class="&rdquo;brush:"> # ---------------------------------------------------
# Script: C:\Scripts\StopAzureVM.ps1
# Version: 0.1
# Author: Stefan Stranger
# Date: 11/18/2016 16:06:47
# Description: Stop Azure VM PowerShell Script Runbook triggered by WebHook
# Comments:
# Changes:  
# Disclaimer: 
# This example is provided "AS IS" with no warranty expressed or implied. Run at your own risk. 
# **Always test in your lab first**  Do this at your own risk!! 
# The author will not be held responsible for any damage you incur when making these changes!
# ---------------------------------------------------

&lt;# 
    The values ResourceGroup name and VM Name are sent in the body response from the webrequest.
    The body should look like this:
    @{ ResourcegroupName="ResourceGroupName";VMName="SimpleWindowsVM"}
#&gt;
[CmdletBinding()]
param (
        [object]$WebhookData
    )

#Verify if Runbook is started from Webhook.

# If runbook was called from Webhook, WebhookData will not be null.
if ($WebhookData){

    # Collect properties of WebhookData
    $WebhookName     =     $WebhookData.WebhookName
    $WebhookHeaders =     $WebhookData.RequestHeader
    $WebhookBody     =     $WebhookData.RequestBody

    # Collect individual headers. VMList converted from JSON.
    $From = $WebhookHeaders.From
    $VMInfo = ConvertFrom-Json -InputObject $WebhookBody
    Write-Output -InputObject ('Runbook started from webhook {0} by {1}.' -f $WebhookName, $From)
    }
else
    {
        Write-Error -Message 'Runbook was not started from Webhook' -ErrorAction stop
    }



#region Variables
$connectionName = 'AzureRunAsConnection'
#endregion

#region Connect to Azure
try
{
       
    # Get the connection "AzureRunAsConnection "
    $servicePrincipalConnection=Get-AutomationConnection -Name $connectionName         

    'Logging in to Azure...'
    Add-AzureRmAccount `
        -ServicePrincipal `
        -TenantId $servicePrincipalConnection.TenantId `
        -ApplicationId $servicePrincipalConnection.ApplicationId `
        -CertificateThumbprint $servicePrincipalConnection.CertificateThumbprint 
}
catch 
{
    if (!$servicePrincipalConnection)
    {
        $ErrorMessage = ('Connection {0} not found.' -f $connectionName)
        throw $ErrorMessage
    } else{
        Write-Error -Message $_.Exception
        throw $_.Exception
    }
}
#endregion


#region shutdown vm
#Retrieve VM with Status information
$VM = Get-AzureRmVM -ResourceGroupName $($VMInfo.ResourceGroupName) -Name $($VMInfo.VMName) -Status 
if ($vm.statuses | where-object {$_.code -eq 'PowerState/deallocated' }) {
    Write-output -InputObject ('VM {0} is already in PowerState Deallocated' -f $VM.name)
}
else
{
    Write-output -InputObject ('Shutting down VM {0}' -f $VM.name)
    $VM | Stop-AzureRMVM -Force
}
#endregion
</pre>
<p><strong></strong>The final step is calling the runbook using the webhook you created earlier. We configure the following in <strong>HttpMaster</strong></p>
<p>Url: url from the webhook you created for your runbook in the Azure Portal.</p>
<p>Body:</p><pre class="&rdquo;brush:">{ 
   "ResourcegroupName":"[ResourceGroupName]",
   "VMName":"[NameOfVMtoStop]"
}
</pre>
<p>&nbsp;</p>
<p><a href="https://msdnshared.blob.core.windows.net/media/2016/11/image465.png"><img title="image" style="border-left-width: 0px;border-right-width: 0px;border-bottom-width: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border-top-width: 0px" border="0" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb380.png" width="608" height="572"></a></p>
<p>In the header you can configure the following information:</p><pre class="&rdquo;brush:">From:stefan@contoso.com";Date="11/19/2016 15:47:00"
</pre>
<p><strong></strong>&nbsp;<a href="https://msdnshared.blob.core.windows.net/media/2016/11/image466.png"><img title="image" style="border-left-width: 0px;border-right-width: 0px;border-bottom-width: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border-top-width: 0px" border="0" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb381.png" width="608" height="572"></a></p>
<p><strong></strong>You should now be able to Stop the VM using the web request.</p>
<p><a href="https://msdnshared.blob.core.windows.net/media/2016/11/image467.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;margin: 0px;padding-right: 0px" border="0" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb382.png" width="885" height="176"></a></p>
<p><a href="https://msdnshared.blob.core.windows.net/media/2016/11/image468.png"><img title="image" style="border-left-width: 0px;border-right-width: 0px;border-bottom-width: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border-top-width: 0px" border="0" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb383.png" width="885" height="343"></a></p>
<p><a href="https://msdnshared.blob.core.windows.net/media/2016/11/image469.png"><img title="image" style="border-left-width: 0px;border-right-width: 0px;border-bottom-width: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border-top-width: 0px" border="0" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb384.png" width="885" height="189"></a></p>
<p><a href="https://msdnshared.blob.core.windows.net/media/2016/11/image470.png"><img title="image" style="border-left-width: 0px;border-right-width: 0px;border-bottom-width: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border-top-width: 0px" border="0" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb385.png" width="885" height="350"></a></p>
<p><a href="https://msdnshared.blob.core.windows.net/media/2016/11/image471.png"><img title="image" style="border-left-width: 0px;border-right-width: 0px;border-bottom-width: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border-top-width: 0px" border="0" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb386.png" width="854" height="313"></a></p>
<p>I hope you enjoyed these blog series about using the Azure REST API.</p>
<p>You can find the PowerShell scripts and HttpMaster files at the following Github Repository: <a title="https://github.com/stefanstranger/AzureARMRESTAPI" href="https://github.com/stefanstranger/AzureARMRESTAPI">https://github.com/stefanstranger/AzureARMRESTAPI</a></p>
<p><strong>References:</strong></p>
<ul>
<li><a href="https://github.com/Azure/azure-quickstart-templates" target="_blank">Azure Quickstart templates on Github</a>. 
<li><a href="https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows" target="_blank">Very simple simple deployment of an Windows VM.</a> 
<li><a href="https://msdn.microsoft.com/en-us/library/azure/mt662285.aspx" target="_blank">Microsoft Azure Automation API</a> 
<li><a href="https://docs.microsoft.com/en-us/azure/automation/automation-webhooks" target="_blank">Azure Automation webhooks</a>
<li><a href="https://github.com/stefanstranger/AzureARMRESTAPI" target="_blank">Github Repository</a>
<li><a href="https://blogs.technet.microsoft.com/stefan_stranger/2016/10/21/using-the-azure-arm-rest-apin-get-access-token/" target="_blank">Using the Azure ARM REST API – Get Access Token</a>
<li><a href="https://blogs.technet.microsoft.com/stefan_stranger/2016/10/29/using-the-azure-arm-rest-api" target="_blank">Using the Azure ARM REST API – Get Subscription Information</a>
<li><a href="https://blogs.technet.microsoft.com/stefan_stranger/2016/11/04/using-the-azure-arm-rest-api-end-to-end-example-part-1" target="_blank">Using the Azure ARM REST API – End to end Example Part 1</a></li></ul>
