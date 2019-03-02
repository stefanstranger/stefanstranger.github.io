---
layout: post
title: Managed Identity in Azure DevOps Service Connections
categories: [IaM, Azure, DevOps]
tags: [IaM, Azure, DevOps]
comments: true
---

I reciently noticed that there is a now an option to use **Managed Identity Authentication** for Azure DevOps Connection Services besides Service Principal Authentication.


For those not familair with **<a href="https://docs.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops" target="_blank">Azure DevOps Connection Services</a>**, you use them to connect to external and remote services to execute tasks for a build or deployment.

In this blog post I'm going to explain how to use Managed Identity Authentication for the Azure DevOps Connection Service.

## Managed Identity
What is Managed Identity (formaly know as Managed Service Identity)? 

It's a feature in Azure Active Directory that provides Azure services with an automatically managed identity. You can use this identity to authenticate to any service that supports Azure AD authentication without having any credentials in your code.

Managed Identities only allows an Azure Service to request an Azure AD bearer token.

The here are two types of managed identities:
* A system-assigned managed identity is enabled directly on an Azure service instance. When the identity is enabled, Azure creates an identity for the instance in the Azure AD tenant that's trusted by the subscription of the instance. After the identity is created, the credentials are provisioned onto the instance. The lifecycle of a system-assigned identity is directly tied to the Azure service instance that it's enabled on. If the instance is deleted, Azure automatically cleans up the credentials and the identity in Azure AD.

* A user-assigned managed identity is created as a standalone Azure resource. Through a create process, Azure creates an identity in the Azure AD tenant that's trusted by the subscription in use. After the identity is created, the identity can be assigned to one or more Azure service instances. The lifecycle of a user-assigned identity is managed separately from the lifecycle of the Azure service instances to which it's assigned.


## Why use Managed Identity?
A common challenge when building cloud applications is how to manage the credentials in your code for authenticating to cloud services. Keeping the credentials secure is an important task. Ideally, the credentials never appear on developer workstations and aren't checked into source control. Azure Key Vault provides a way to securely store credentials, secrets, and other keys, but your code has to authenticate to Key Vault to retrieve them. 
The managed identities for Azure resources feature in Azure Active Directory (Azure AD) solves this problem.

If you use the Managed Identity enabled on a (Windows) Virtual Machine in Azure you can only request an Azure AD bearer token from that Virtual Machine, unlike a Service Principal.



## Steps to use a Service Connection with Managed Identity
High-level you need to execute the following steps:
1. Create a Service Connection of the type Azure Resource Manager with Managed Identity authentication
2. Create an Azure Virtual Machine private Azure DevOps Agent
3. Enable Managed Identity on Azure Virtual Machine
4. Authorize the Managed Identity
5. Configure the Managed Identity Service Connection in your pipelines


### Step 1. Create a Service Connection of the type Azure Resource Manager with Managed Identity authentication

Open your Azure DevOps Project Settings and select Service Connections, and select New service connection.

![](/assets/2019-02-07-02.png)


Select type of Service Connection (Azure Resource Manager) and select Managed Identity Authentication. Enter a Connection name, Subscription ID, Subscription name and Tenant ID.

![](/assets/2019-02-07-01.png)

After the creation of the Service Connection we need to create the Azure Virtual Machine private build agent.

### Step 2. Create an Azure Virtual Machine private Azure DevOps Agent

An agent that you set up and manage on your own to run build and deployment jobs is called a self-hosted agent. You can use self-hosted agents in Azure Pipelines or Team Foundation Server (TFS). Self-hosted agents give you more control to install dependent software needed for your builds and deployments. Also, machine-level caches and configuration persist from run to run, which can boost speed.

You can install the agent on Linux, macOS, or Windows machines. You can also install an agent on a Linux Docker container.

We are going to create a Windows private Azure DevOps Agent (self-hosted Windows Agent).

The Microsoft Azure DevOps team uses <a href="https://www.packer.io/docs/builders/azure-setup.html" target="_blank">Packer</a> to build Azure Virtual Machine images that are being used to create Azure Virtual Machine in Hosted Agent Pools. You can view the Packer config that Microsoft uses at <a href="https://github.com/Microsoft/vsts-image-generation" target="_blank">https://github.com/Microsoft/vsts-image-generation</a> where the whole Agent config is open source.

<a href="https://www.packer.io/intro/index.html" target="_blank">Packer</a> is an open source tool for creating identical machine images for multiple platforms from a single source configuration. Packer is lightweight, runs on every major operating system, and is highly performant, creating machine images for multiple platforms in parallel.

<a href="https://twitter.com/wouterdekort" target="_blank">Wouter de Kort</a> wrote a great blog series about how to <a href="https://wouterdekort.com/2018/02/25/build-your-own-hosted-vsts-agent-cloud-part-1-build/" target="_blank">Build your own Hosted VSTS Agent Cloud</a>. Please look at the References section for all his blog posts. 

You can also find more information on create Azure Virtual Machine images using Packer on <a href="https://docs.microsoft.com/en-us/azure/virtual-machines/windows/build-image-with-packer" target="_blank">Microsoft Docs</a>.

#### Install Packer
You can install Packer on your local development machine using <a href="https://chocolatey.org/" target="_blank">Chocolatey</a>, which is a Windows Package Manager like apt-get on Linux. Open elevated PowerShell host and run:

```powershell
choco install packer
```

#### Create Azure Resource Group
During the build process, Packer creates temporary Azure resources as it builds the source Virtual Machine. To capture that source Virtual Machine for use as an image, you must define a resource group. The output from the Packer build process is stored in this Azure Resource Group.

Run the following PowerShell commands from a PowerShell host after logging into Azure.

```powershell
$ResoureGroupName = "[Enter Resource Group Name]"
$Location = "westeurope"
New-AzureRmResourceGroup -Name $ResourceGroupName -Location $Location
```

#### Create Azure credentials
Packer authenticates with Azure using a service principal (now also Managed Identity is supported). An Azure service principal is a security identity that you can use with apps, services, and automation tools like Packer. You control and define the permissions as to what operations the service principal can perform in Azure.

Run the following PowerShell commands from a PowerShell host after logging into Azure.

```powershell
$ServicePrincipal = New-AzureRmADServicePrincipal -DisplayName "[Enter a Name for the Azure Packer Service Principal]" `
    -Password (ConvertTo-SecureString "[Enter password]" -AsPlainText -Force)
Sleep 20
New-AzureRmRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $ServicePrincipal.ApplicationId
```

To authenticate to Azure, you also need to obtain your Azure tenant and subscription IDs with Get-AzureRmSubscription.

Run the following PowerShell commands from a PowerShell host after logging into Azure.

```powershell
$sub = Get-AzureRmSubscription -SubscriptionName "[Enter SubscriptionName]"
$sub.TenantId
$sub.SubscriptionId
```

You need to use the Azure Tenantid and SubscriptionId values in the Packer Settings file you are creating in the next steps. So save them some somewhere.

#### Packer Build
We can use Packer to run a local build to create an Azure Virtual Machine image. If we look at the help information from Packer we see that the following commands can be used.

```bash
packer --help
Usage: packer [--version] [--help] <command> [<args>]

Available commands are:
    build       build image(s) from template
    fix         fixes templates from old versions of packer
    inspect     see components of a template
    validate    check that a template is valid
    version     Prints the Packer version
```

For the build step the following arguments are needed.

```bash
packer build --help
Usage: packer build [options] TEMPLATE

  Will execute multiple builds in parallel as defined in the template.
  The various artifacts created by the template will be outputted.

Options:

  -color=false                  Disable color output. (Default: color)
  -debug                        Debug mode enabled for builds.
  -except=foo,bar,baz           Build all builds other than these.
  -only=foo,bar,baz             Build only the specified builds.
  -force                        Force a build to continue if artifacts exist, deletes existing artifacts.
  -machine-readable             Produce machine-readable output.
  -on-error=[cleanup|abort|ask] If the build fails do: clean up (default), abort, or ask.
  -parallel=false               Disable parallelization. (Default: parallel)
  -timestamp-ui                 Enable prefixing of each ui output with an RFC3339 timestamp.
  -var 'key=value'              Variable for templates, can be used multiple times.
  -var-file=path                JSON file containing user variables.
```

The packer build command takes a template and runs all the builds within it in order to generate a set of artifacts. The various builds specified within a template are executed in parallel, unless otherwise specified. And the artifacts that are created will be outputted at the end of the build.

Packer build needs at the minimum two files, a Packer settings file (-var-file) and JSON template file. Templates are JSON files that configure the various components of Packer in order to create one or more machine images. Templates are portable, static, and readable and writable by both humans and computers.

Example Packer Settings file (packersettings.json)

```json
{
    "client_id": "[Enter Application Id of Service Principal]",
    "client_secret": "[Enter Password Service Principal]",
    "tenant_id": "[Enter the Subscription Tenant ID]",
    "subscription_id": "[Enter Subscription ID]",
    "location": "[Enter Location of Resource Group. Example 'WestEurope']",
    "managed_image_resource_group_name": "[Enter Resource Group Name where to store the Azure Virtual Machine image]",
    "managed_image_name": "[Enter Azure Virtual Machine Image name. Example 'windows-image']"
}
```

Save and update the packersettings.json file with your configurations in c:\temp folder.

Example Simple Azure Virtual Machine template (simple-windows.json)
```json
{
    "variables": {
        "client_id": "{{env `ARM_CLIENT_ID`}}",
        "client_secret": "{{env `ARM_CLIENT_SECRET`}}",
        "subscription_id": "{{env `ARM_SUBSCRIPTION_ID`}}",
        "tenant_id": "{{env `ARM_TENANT_ID`}}",
        "object_id": "{{env `ARM_OBJECT_ID`}}",
        "location": "{{env `ARM_RESOURCE_LOCATION`}}",
        "managed_image_resource_group_name": "{{env `ARM_IMAGE_RESOURCE_GROUP_NAME`}}",
        "managed_image_name": "{{env `ARM_IMAGE_NAME`}}"
    },
    "builders": [{
        "type": "azure-arm",
        "client_id": "{{user `client_id`}}",
        "client_secret": "{{user `client_secret`}}",
        "subscription_id": "{{user `subscription_id`}}",
        "object_id": "{{user `object_id`}}",
        "tenant_id": "{{user `tenant_id`}}",
        "location": "{{user `location`}}",
        "vm_size": "{{user `vm_size`}}",
        "managed_image_resource_group_name": "{{user  `managed_image_resource_group_name`}}",
        "managed_image_name": "{{user `managed_image_name`}}",
        "os_type": "Windows",
        "image_publisher": "MicrosoftWindowsServer",
        "image_offer": "WindowsServer",
        "image_sku": "2016-Datacenter",
        "communicator": "winrm",
        "winrm_use_ssl": "true",
        "winrm_insecure": "true",
        "winrm_timeout": "4h",
        "winrm_username": "packer"
    }],
    "provisioners": [{
        "type": "powershell",
        "inline": [
            "if( Test-Path $Env:SystemRoot\\windows\\system32\\Sysprep\\unattend.xml ){ rm $Env:SystemRoot\\windows\\system32\\Sysprep\\unattend.xml -Force}",
            "& $env:SystemRoot\\System32\\Sysprep\\Sysprep.exe /oobe /generalize /quiet /quit",
            "while($true) { $imageState = Get-ItemProperty HKLM:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Setup\\State | Select ImageState; if($imageState.ImageState -ne 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { Write-Output $imageState.ImageState; Start-Sleep -s 10  } else { break } }"
        ]
    }]
}
```

Save above simple-server.json file in c:\temp folder.

With this Packer Template file a Windows Server 2016 Azure Virtual Machine image is being created.

To build and deploy this Windows Server 2016 Virtual Image to Azure run the following Packer build command from your PowerShell host.
```PowerShell
packer build -debug -on-error=ask -var-file=C:\temp\packersettings.json C:\temp\simple-windows.json
```
Using the debug and on-error parameters you get some more overview of the Packer build and deployment steps in Azure. 

Be patient it can take up to 20 minutes before the Windows Server 2016 Azure Virtual Image is being created.

If everything works as expected you would see the following Virtual Image created in your configured Azure Resource Group.

![](/assets/2019-02-07-03.png)


#### Create Windows Server 2016 from Azure Virtual Machine Image
From the Azure Virtual Machine image create an Azure Virtual Machine by selecting create VM in the Virtual Image Pane and go through the configuration options. 

![](/assets/2019-02-07-04.png)

**Remarks:**
* When selecting a VM Size make sure you have enough RAM (1 GB RAM is not enough)
* To be able to install the Azure DevOps Agent manually, enable RDP as Public inbound port.
  Please be aware that this port will be exposed to the internet!

![](/assets/2019-02-07-05.png)

The end result should be a Windows Server 2016 Azure Virtual Machine in your Resource Group.

#### Install the Azure DevOps Agent
We are manually going to install the Azure DevOps Agent on the Virtual Machine deployed in the previous steps. 

Before continuing check the prerequisites for the <a href="https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/v2-windows?view=azure-devops" target="_blank">Self-hosted Windows Agents</a>. For now you should be good to go. 

For configuring the Azure DevOps Agent we need to have a Personal Access Token (PAT) from the Azure DevOps Project where we want to use the private Build Agents.

Follow the steps documented <a href="https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/v2-windows?view=azure-devops#authenticate-with-a-personal-access-token-pat" target="_blank">here</a> to create the PAT.

#### Connect to Azure Virtual Machine and install Azure DevOps Agent

In the next step we are going to install the Azure DevOps Agent manually by connecting to the Azure Virtual Machine via RDP. 

Select the Connect button in the Azure Portal to connect to the Azure Virtual Machine.

![](/assets/2019-02-07-06.png)


![](/assets/2019-02-07-07.png)

Download the RDP file and connect with the useraccount and password you configured earlier.

1. In your web browser, sign in to Azure Pipelines, and navigate to the Agent pools tab: 
2. Azure Pipelines: https://dev.azure.com/{your_organization}/_admin/_AgentPool
3. Click Download agent.

    ![](/assets/2019-02-07-08.png)

4. On the Get agent dialog box, click Windows.
5. On the left pane, select the processor architecture of your machine (x86 or x64). 
6. On the right pane, click the Download button. 
7. Follow the instructions on the page to download the agent.
8. Unpack the agent into the directory of your choice. Then run config.cmd.
   
   Make sure you have the Azure DevOps Server URL and Personal Access Token available when running the config.cmd on the Azure Virtual Machine.

    ![](/assets/2019-02-07-09.png)



Install Azure AZ PowerShell modules on Self-Hosted Agent in Azure. We need these modules for the PowerShell Task script we want to run in the Azure DevOps Pipelines.

**Remark:**

At the moment of writing this blog article the <a href="https://github.com/Microsoft/azure-pipelines-tasks/issues/9201" target="_blank">Azure PowerShell Tasks didn't support PowerShell AZ Modules ye</a>t. So we need to authenticate against Azure within the PowerShell script used in the PowerShell task.

Run the following PowerShell command on the Self-Hosted Agent Azure Virtual Machine.

```powershell
Install-Module -Name Az -Scope AllUsers
```

We are going to use the Azure Az PowerShell modules within the PowerShell Tasks of the Azure DevOps Pipelines.

### Step 4. Enable Managed Identity on Azure Virtual Machine
Go to the Azure Portal and go the Windows Virtual Machine you deployed in step 2. and select Identity and change the status to on.

![](/assets/2019-02-07-11.png)

If you copy the Object ID you can the check the Principal Key Credentials with PowerShell.

```powershell
Get-AzureADServicePrincipalKeyCredential -ObjectId "[Enter Object ID]" -OutVariable MICredential
New-TimeSpan -Start $MICredential.StartDate -End $MICredential.EndDate | Select Days
```

![](/assets/2019-02-07-12.png)


### Step 4. Authorize the Managed Identity

We are authorizing the Virtual Machine Identity on an already existing Key Vault located in another Resource Group then where the Virtual Machine is deployed.

Go to the Key Vault Resource in the Azure Portal and authorize the Managed Identity.

![](/assets/2019-02-07-13.png)


![](/assets/2019-02-07-14.png)

### Step 5. Configure the Managed Identity Service Connection in your pipelines

In the last step we are going to create a Release Pipeline using the self-hosted Windows Virtual Machine configured as Managed Identity.

Go to your Azure DevOps Project and select Releases under Pipelines and create a new Release.

![](/assets/2019-02-07-15.png)

Configure the Agent job to use the Private (Default) Agent Pool.

![](/assets/2019-02-07-16.png)

Add a PowerShell Task to run a PowerShell script on the self-hosted Windows Virtual Machine.

![](/assets/2019-02-07-17.png)

With this task we are going to retrieve the properties for the Key Vault for which we authenticated the Managed Identity.

Use the following PowerShell script:

```powershell
#region get Access Token
$response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -UseBasicParsing -Method GET -Headers @{Metadata="true"}
$content = $response.Content | ConvertFrom-Json
$AccessToken = $content.access_token
#endregion

#region login to Azure with Access Token
Add-AzAccount -Tenant '[Enter tenantid]' -Subscription '[Enter SubscriptionId]' -KeyVaultAccessToken $AccessToken -AccessToken $AccessToken -AccountId 'ManagedIdentity;'
#endregion

#region Get Key Vault Resource
Get-AzKeyVault -VaultName "[Enter Key Vault Name]"" -ResourceGroupName "[Enter Resource Group Name where Key Vault is stored]"
#endregion
```

## Azure Instance Metadata Service

To retrieve the Managed Identity Access Token I used the <a href="https://docs.microsoft.com/en-us/azure/virtual-machines/windows/instance-metadata-service" target="_blank">Azure Instance Metadata Service</a>.

The Azure Instance Metadata Service provides information about running virtual machine instances that can be used to manage and configure your virtual machines. This includes information such as SKU, network configuration, and upcoming maintenance events. For more information on what type of information is available, see metadata categories.

Azure's Instance Metadata Service is a REST Endpoint accessible to all IaaS VMs created via the Azure Resource Manager. The endpoint is available at a well-known non-routable IP address (169.254.169.254) that can be accessed only from within the VM.

When we run the release with this task we should be able to retrieve the properties of the Key Vault.

![](/assets/2019-02-07-18.png)

If we want to get some more information about the Access Token we can use the <a href="https://github.com/stefanstranger/psjwt" target="_blank">PowerShell psjwt Module</a>.

We can use this module in a PowerShell Task in our Release Pipeline. Create a new PowerShell task in your Release Pipeline.

![](/assets/2019-02-07-19.png)

Use the following PowerShell code as inline script:

```powershell
#region install psjwt module from PSGallery
Install-PackageProvider -Name NuGet -Force -Scope CurrentUser
Install-Module -Name psjwt -Force -Scope Currentuser
#endregion

#region retrieve Access Token
$response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -UseBasicParsing -Method GET -Headers @{Metadata="true"}

$content = $response.Content | ConvertFrom-Json

$AccessToken = $content.access_token
#endregion

#region Convert JWT Access Token
Convertfrom-Jwt -Token $AccessToken
#endregion
```
![](/assets/2019-02-07-20.png)

The result of above PowerShell task.

If you are interested you can find more information about Azure Active Directory ID token headers <a href="https://docs.microsoft.com/en-us/azure/active-directory/develop/id-tokens#header-claims" target="_blank">here</a>.

Hope you learned how to start using Managed Identity in your Azure DevOps Connections.



**References:**
* <a href="https://docs.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops" target="_blank">Service connections for builds and releases</a>
* <a href="https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/" target="_blank">Azure AD managed identities for Azure resources documentation</a>
* <a href="https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview" target="_blank">What is managed identities for Azure resources?</a>
* <a href="https://app.pluralsight.com/library/courses/microsoft-azure-resources-managed-identities-implementing/table-of-contents" target="_blank">Pluralsight - Implementing Managed Identities for Microsoft Azure Resources</a>
* <a href="https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/v2-windows?view=azure-devops" target="_blank">Selfhosted Windows Agents</a>
* <a href="https://www.packer.io/intro/index.html" target="_blank">Introduction to Packer</a>
* <a href="https://wouterdekort.com/2018/02/25/build-your-own-hosted-vsts-agent-cloud-part-1-build/" target="_blank">Build your own Hosted VSTS Agent Cloud: Part 1 – Build</a>
* <a href="https://wouterdekort.com/2018/02/27/build-your-own-hosted-vsts-agent-cloud-part-2-deploy/" target="_blank">Build your own Hosted VSTS Agent Cloud: Part 2 – Deploy</a>
* <a href="https://wouterdekort.com/2018/03/05/build-hosted-vsts-agent-cloud-part-3-automate/" target="_blank">Build your own Hosted VSTS Agent Cloud: Part 3 – Automate</a>
* <a href="https://wouterdekort.com/2018/03/13/build-your-own-hosted-vsts-agent-cloud-part-4-customize/" target="_blank">Build your own Hosted VSTS Agent Cloud: Part 4 – Customize</a>
* <a href="https://blog.kloud.com.au/2018/04/13/demystifying-managed-service-identities-on-azure/" target="_blank">Blog post - Demystifying Managed Service Identities on Azure</a>
* <a href="https://docs.microsoft.com/en-us/azure/virtual-machines/windows/instance-metadata-service" target="_blank">Azure Instance Metadata service</a>
