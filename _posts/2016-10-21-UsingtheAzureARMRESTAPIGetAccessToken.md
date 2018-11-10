---
layout: post
title: Using the Azure ARM REST API – Get Access Token
categories: [Azure, PowerShell]
tags: [Azure, PowerShell]
comments: true
---
This week I’ve been busy with trying to figure out how you can ‘directly’ talk to the Azure ARM REST API instead of using PowerShell or the Azure CLI. Because I could not find a lot of information about this topic online I thought it would nice to share some of learnings.

But why would you even want to directly talk to the Azure ARM REST API? Good question  Most of the time I would recommend using tools like PowerShell or the Azure CLI to communicate with the Azure ARM REST API because that’s often way easier. In this case the customer wanted to have all the workflow logic centralized in the tooling which was used for the deployment of the Azure Resources. The deployment tooling could deploy an ARM Template but for the complete configuration of the Azure Resource (WebApp) there was also a need for some pre- and post-activities like configuration of the Diagnostic Logging which preferably should be done using ‘simple’ web service calls to the Azure ARM REST API.
In this firs blog post I’m going to describe how you could get the AccessToken needed for the further Authentication against the Azure ARM REST API.

**Azure ARM REST API**

Azure Resource Manager provides a new way for you to deploy and manage the services that make up your applications. For an introduction to deploying and managing resources with Resource Manager, see <a href="https://azure.microsoft.com/documentation/articles/resource-group-overview/" target="_blank">Azure Resource Manager Overview</a>. Most, but not all, services support Resource Manager, and some services support Resource Manager only partially. Microsoft will enable Resource Manager for every service that is important for future solutions, but until the support is consistent, you need to know the current status for each service. For information about the available services and how to work with them, see <a href="https://azure.microsoft.com/documentation/articles/resource-manager-supported-services/" target="_blank">Resource Manager providers, regions, API versions and schemas</a>. [*from <a href="https://msdn.microsoft.com/en-us/library/azure/dn790568.aspx" target="_blank">Azure Resource Manager REST API Reference</a>]

**Authentication**
So how does the authentication work when you want to to do a web request call against the Azure ARM REST API? You need to supply a bearer Access Token in the request Header of the web request. But how do you get that AccessToken? You can retrieve the AccessToken by creating an Active Directory application and service principal and use a ClientID and ClientSecret to retrieve the AccessToken. We will use PowerShell to create the Service Principal to access resources in Azure.

**Create a service principal to access resources:**

1. Create the AD application with a password
2. Create the service principal
3. Assign the Contributor role to the service principal

I used the following PowerShell code:

```powershell
#Login to Azure
Add-AzureRmAccount
 
#Select Azure Subscription
$subscription = 
    (Get-AzureRmSubscription |
        Out-GridView `
        -Title 'Select an Azure Subscription ...' `
    -PassThru)
 
Set-AzureRmContext -SubscriptionId $subscription.subscriptionId -TenantId $subscription.TenantID

#create SPN with Password
New-AzureRmADApplication -DisplayName "demowebrequest" -HomePage "https://www.stranger.nl/demowebrequest" -IdentifierUris "https://www.stranger.nl/demowebrequest" -Password "P@ssw0rd!" -OutVariable app
New-AzureRmADServicePrincipal -ApplicationId $app.ApplicationId
New-AzureRmRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $app.ApplicationId.Guid

Get-AzureRmADApplication -DisplayNameStartWith 'demowebrequest' -OutVariable app
Get-AzureRmADServicePrincipal -ServicePrincipalName $app.ApplicationId.Guid -OutVariable SPN
```
![](/assets/restapiblog1-1.png)

If you now go to App Registrations in the Azure Portal you see the demowebrequest application being created.

![](/assets/restapiblog1-2.png)

We now need to create a new Service Principal Name and assign the correct Contributor Role.

![](/assets/restapiblog1-3.png)

If everything goes ok you see the following in the Azure Portal under App Registrations –> demorequest –> Settings.

![](/assets/restapiblog1-4.png)

Next we need to set the correct **Required permissions** and create a **Key**.
Set required permissions. Go to Required Permissions and click on Add.

![](/assets/restapiblog1-5.png)

Select the Access Azure Service Management as organization users (preview) API.

![](/assets/restapiblog1-6.png)

And finally select the following permissions.

![](/assets/restapiblog1-9.png)

We have now configured the correct permissions for the application..

![](/assets/restapiblog1-10.png)

The last step in this process is to create a new Key.
Select Keys under App registrations –> [appname] –> Settings pane in the Azure Portal and create a new key.

![](/assets/restapiblog1-11.png)

Enter a Key description and save the value on save.

![](/assets/restapiblog1-12.png)

We now have the following information available to get an AccessToken:
* ClientId: this is application id which can be found in the Azure Portal

![](/assets/restapiblog1-13.png)

* ClientSecret: this is the key value which we created earlier.

**Use ClientId and ClientSecret to retrieve AccessToken**

Now we have the ClientID and ClientSecret we can do web call to receive an AccessToken which can be used for authentication against the Azure ARM REST API.
Let’s use CURL to retrieve the AccessToken. You also need to enter the tennantid in the request url. You can find the tennantid if you have use the PowerShell script I showed earlier by returning the $subscription.tennantid value in PowerShell.

```bash
curl --request POST "https://login.windows.net/[tennantid]/oauth2/token" --data-urlencode "resource=https://management.core.windows.net" --data-urlencode "client_id=[clientid]" --data-urlencode "grant_type=client_credentials" --data-urlencode "client_secret=[clientsecret]"
```

![](/assets/restapiblog1-14.png)


If you would rather use PowerShell to retrieve this AccesToken you can use the following PowerShell code:

```powershell
#Azure Authtentication Token

#requires -Version 3
#SPN ClientId and Secret
$ClientID       = "clientid" #ApplicationID
$ClientSecret   = "ClientSecret"  #key from Application
$tennantid      = "TennantID"
 

$TokenEndpoint = {https://login.windows.net/{0}/oauth2/token} -f $tennantid 
$ARMResource = "https://management.core.windows.net/";

$Body = @{
        'resource'= $ARMResource
        'client_id' = $ClientID
        'grant_type' = 'client_credentials'
        'client_secret' = $ClientSecret
}

$params = @{
    ContentType = 'application/x-www-form-urlencoded'
    Headers = @{'accept'='application/json'}
    Body = $Body
    Method = 'Post'
    URI = $TokenEndpoint
}

$token = Invoke-RestMethod @params

$token | select access_token, @{L='Expires';E={[timezone]::CurrentTimeZone.ToLocalTime(([datetime]'1/1/1970').AddSeconds($_.expires_on))}} | fl *
```
![](/assets/restapiblog1-15.png)

In the next blog post we are going to use this AccessToken to authenticate against the Azure ARM REST API and do some more web requests.

Hope you like it.
 
**References:**
* <a href="https://msdn.microsoft.com/en-us/library/azure/dn790568.aspx" target="_blank">Azure Resource Manager REST API Reference</a>
* <a href="https://azure.microsoft.com/en-us/documentation/articles/resource-group-authenticate-service-principal/" target="_blank">Use Azure PowerShell to create a service principal to access resources</a>
