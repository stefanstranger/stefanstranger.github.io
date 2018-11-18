---
layout: post
title: Using the Windows Graph API to set Service Principal as owner of Service Principal
categories: [PowerShell, Azure Active Directory]
tags: [PowerShell, Azure]
comments: true
---

In the blog post I'll explain how to use the 'old' Windows Graph API to set an Azure Service Principal as owner of another Service Principal.

For the automated configuration of Azure DevOps service connections for DevOps teams we use a "Parent" Service Principal which creates "Child" Service Principals which are being used to create a service connection for an Azure Subscription.

This "Child" Service Principal will be used to connect to Azure. The service principal specifies the resources and the access levels that will be available over the connection. 

When you register an Azure AD application in the Azure portal, two objects are created in your Azure AD tenant:
* An application object, and
* A service principal object

### Application object
An Azure AD application is defined by its one and only application object, which resides in the Azure AD tenant where the application was registered, known as the application's "home" tenant. The Azure AD Graph <a href="https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#application-entity" target="_blank">Application entity</a> defines the schema for an application object's properties.
### Service principal object
To access resources that are secured by an Azure AD tenant, the entity that requires access must be represented by a security principal. This is true for both users (user principal) and applications (service principal).
The security principal defines the access policy and permissions for the user/application in the Azure AD tenant. This enables core features such as authentication of the user/application during sign-in, and authorization during resource access.
When an application is given permission to access resources in a tenant (upon registration or <a href="https://docs.microsoft.com/en-us/azure/active-directory/develop/developer-glossary#consent" target="_blank">consent</a>), a service principal object is created. The Azure AD Graph <a href="https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#serviceprincipal-entity" target="_blank">ServicePrincipal entity</a> defines the schema for a service principal object's properties.

## Situation
Quite some Service Principals being used in the Service Connection in Azure DevOps Pipelines had an old owner configured and needed to have the "Parent" Service Principal as a new owner.

I initially used the following PowerShell code to set the "Parent" Service Principal as owner for the "Child" Service Principal.

```powershell
#region import module
Import-Module AzureAD
#endregion

#region variables
$tenantid = '[enter tenant id]' 
#endregion

#region Connect-AzureAD
Connect-AzureAD -TenantId $tenantid
#endregion

#region Get Current Session info
Get-AzureADCurrentSessionInfo
#endregion

#region Get Service Principals
Get-AzureADServicePrincipal -SearchString 'Parent' -OutVariable Parent
Get-AzureADServicePrincipal -SearchString 'Child-01' -OutVariable Child
#endregion

#region Set Parent SP as Owner for Child SP
Add-AzureADServicePrincipalOwner -ObjectId $Child.ObjectId -RefObjectId $Parent.ObjectId
#endregion

#region Check owner for Child Service Principal
Get-AzureADServicePrincipalOwner -All $true -ObjectId $Child.ObjectId
#endregion
```

This worked as expected but I wanted to learn more about using the Windows Graph API so I tried to do same as above but now directly 'talking' to the Windows Graph API.

**Remark:**


<table>
<tbody>
	<tr>
		<td>
		Microsoft strongly recommends that you use <a href="https://developer.microsoft.com/graph/" target="_blank">Microsoft Graph</a> instead of Azure AD Graph API to access Azure Active Directory resources. Our development efforts are now concentrated on Microsoft Graph and no further enhancements are planned for Azure AD Graph API. There are a very limited number of scenarios for which Azure AD Graph API might still be appropriate; for more information, see the Microsoft Graph or the <a href="https://dev.office.com/blogs/microsoft-graph-or-azure-ad-graph" target="_blank">Azure AD Graph blog post</a> in the Office Dev Center.
        </td>
	</tr>
</tbody>
</table>



Why did I then used the Windows Graph API? Because the AzureAD module is also using the Windows Graph API.

## Authentication
Before we can 'talk' to the Azure AD Graph we first need to authenticate. If you want to interactively authenticate with Azure AD Graph the easiest way is to use the AzureAD PowerShell module's **Connect-AzureAD** cmdlet.

```powershell
#region import module
Import-Module AzureAD
#endregion

#region variables
$tenantid = '[enter tenant id]' 
#endregion

#region Connect-AzureAD
Connect-AzureAD -TenantId $tenantid
#endregion
```

Enter your credentials in the following pop-up window.

![Sign in Window](/assets/grapapi_1.png)

With the Get-AzureADCurrentSessionInfo cmdlet you can get information about your current Azure AD Session if you want.

## Access Token
For the authorization part against the Azure AD Graph we need to use an Access Token in the Authorization Header of the web request.

You can use the following code to retrieve the Access Token when authenticating with the Connect-AzureAD cmdlet.

```powershell
#region retrieve token
$Token = ([Microsoft.Open.Azure.AD.CommonLibrary.AzureSession]::AccessTokens['AccessToken']).AccessToken
#endregion
```

You can verify JSON Tokens online or on your local machine. JSON Web Tokens are an open, industry standard RFC 7519 method for representing claims securely between two parties.

This time we are going to use JWT-Encode node module to decode the Access Token instead of online tools. <a href="https://github.com/auth0/jwt-decode" target="_blank">Jwt-decode</a> is a small browser library that helps decoding JWTs token which are Base64Url encoded.

We can use this in a Javascript. First install <a href="https://nodejs.org/en/download/" target="_blank">NodeJs</a> if you have not installed that yet. 

I've currently version 10.3.0 installed on my system.

![](/assets/grapapi_2.png)

```powershell
#region create directory
mkdir c:\temp\token
cd c:\temp\token
#endregion

#region install jwt-decode 
npm install jwt-decode
#endregion

#region create javascript to decode Access Token
$javascript = @'
var token = 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJGaXJzdE5hbWUiOiJTdGVmYW4iLCJMYXN0TmFtZSI6IlN0cmFuZ2VyIiwiRGVtbyI6IkVuY29kZSBBY2Nlc3MgVG9rZW4iLCJleHAiOjEzOTMyODY4OTMsImlhdCI6MTM5MzI2ODg5M30.8-YqAPPth3o-C_xO9WFjW5RViAnDe2WrmVyqLRnNEV0'

if (typeof jwt_decode === 'undefined') {
  var jwt_decode = require('./node_modules/jwt-decode');
}
var decoded = jwt_decode(token);

console.log(decoded);
'@

$javascript | Out-File -FilePath c:\temp\token\decodetoken.js -Encoding utf8
#endregion

#region test encodetoken.js
node c:\temp\token\decodetoken.js
```

You should now see the following with the demo token.

![Encoded Access Token](/assets/grapapi_3.png)
 
 
 Let's verify the Access Token from the Connect-AzureAD cmdlet.
 
 ```powershell
#region copy token to clipboard
$Token | clip
#endregion

#insert clipboard value into javascript c:\temp\decodetoken.js

#region run decodetoken javascript
node c:\temp\decodetoken.js
#endregion
```

![](/assets/grapapi_4.png)

We can now continue to use this token (as long as it has not expired) in our PowerShell REST API calls.

With the following PowerShell code we can retrieve the tenant information.

```powershell
#region variables
$tenantid = '[enter tenant id]'
$tenantDomain = '[enter tenant domain name].onmicrosoft.com'
$apiversion = '1.6'
$Resource = 'tenantDetails'
#endregion

#region connect to Azure AD
Connect-AzureAD -TenantId $tenantid
#endregion

#region retrieve token
$Token = ([Microsoft.Open.Azure.AD.CommonLibrary.AzureSession]::AccessTokens['AccessToken']).AccessToken
#endregion

#region Get Tenant info
$Uri = ('https://graph.windows.net/{0}/{1}?api-version={2}' -f $tenantDomain, 'tenantDetails', $apiversion)

$params = @{
    ContentType = 'application/x-www-form-urlencoded'
    Headers     = @{
        'authorization' = "Bearer $($Token)"
    }
    Method      = 'Get'
    URI         = $Uri 
}

$Result = Invoke-RestMethod @params
$Result.value
#endregion
```

With the following function we can retrieve all Service Principals.

```powershell
function Get-ServicePrincipal {
    [CmdletBinding()]
    param (
        [string]$Token,
        [string]$TenantDomain,
        [string]$ApiVersion = '1.6'
    )
    
    begin {
        #region variables
        $Uri = ('https://graph.windows.net/{0}/{1}?api-version={2}' -f $tenantDomain, 'servicePrincipals', $apiversion)
        #endregion
    }
    
    process {

        $params = @{
            ContentType = 'application/json'
            Headers     = @{
                'authorization' = "Bearer $($Token)"
            }
            Method      = 'Get'
            URI         = $Uri 
        }
          
        return (Invoke-RestMethod @params).value
        
    }
}


Get-ServicePrincipal -Token $Token -TenantDomain '[enter tenant domain name].onmicrosoft.com'
```
To retrieve the current owner of a Service Principal we can use the following PowerShell Function:

```powershell
function Get-ServicePrincipalOwner {
    [CmdletBinding()]
    param (
        [string]$Token,
        [string]$ObjectId,
        [string]$TenantDomain,
        [string]$ApiVersion = '1.6'
    )
    
    begin {
        #region variables
        $Uri = ('https://graph.windows.net/{0}/{1}/{2}/owners?api-version={3}' -f $tenantDomain, 'servicePrincipals', $ObjectId, $ApiVersion)
        #endregion
    }
    
    process {

        $params = @{
            ContentType = 'application/json'
            Headers     = @{
                'authorization' = "Bearer $($Token)"
            }
            Method      = 'Get'
            URI         = $Uri 
        }

          
        return (Invoke-RestMethod @params).value
        
    }
    
    end {
    }
}

Get-ServicePrincipalOwner -Token $Token -ObjectId '[object id Child-01 SP]' -TenantDomain '[enter tenant domain name].onmicrosoft.com'
```

![](/assets/grapapi_5.png)

The final step is configuring a (new) Owner for the Child-01 Service Principal.

```powershell
function Set-ServicePrincipalOwner {
    [CmdletBinding()]
    param (
        # ObjectID for Service Principal for which the Owner needs to be set.
        [Parameter(Mandatory = $true,
            ValueFromPipeline = $true)]
        [string]$ObjectId,

        # ObjectId Service Principal Owner.
        [Parameter(Mandatory = $true)]
        [string]
        $ServicePrincipalOwnerObjectId,

        # Tenant Domain Name.
        [Parameter(Mandatory = $true)]
        [string]
        $tenantDomain,

        # Api Version.
        [Parameter(Mandatory = $false)]
        [string]
        $ApiVersion = '1.6'
    )
    
    begin {
        $Uri = ('https://graph.windows.net/{0}/{1}/{2}/$links/owners?api-version={3}' -f $tenantDomain, 'servicePrincipals', $ObjectId, $apiversion)
    }
    
    process {        
 
        $Body = @{
            'url' = ('https://graph.windows.net/{0}/directoryObjects/{1}/Microsoft.DirectoryServices.ServicePrincipal' -f $tenantDomain, $ServicePrincipalOwnerObjectID) 
        } | ConvertTo-Json
 

        $params = @{
            ContentType = 'application/json'
            Headers     = @{
                'authorization' = "Bearer $($Token)"
            }
            Method      = 'Post'
            URI         = $Uri
            Body        = $Body
        }

        Write-Verbose -Message ('Setting Service Principal Owner {0} for Service Principal {1}' -f $ServicePrincipalOwnerObjectId, $ObjectId)
        return (Invoke-RestMethod @params)
    }
    
    end {
    }
}

#region set new Service Principal Owner
Set-ServicePrincipalOwner -ObjectId '[Object Id from Service Princiapl which needs a new owner]d' -ServicePrincipalOwnerObjectID '[Object Id of new Owner]' -tenantDomain '[enter tenant domain name].onmicrosoft.com'
#endregion
```

A new owner is being added when running the Set-ServicePrincipalOwner Function.

![](/assets/grapapi_6.png)

I hoped you learned something new when calling REST API's.

**References:**
* <a href="https://docs.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=vsts" target="_blank">Service connections for builds and releases</a>
* <a href="https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals" target="_blank">Application and service principal objects in Azure Active Directory</a>
* <a href="https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals" target="_blank">Azure AD Graph ServicePrincipal entity</a>
* <a href="https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-graph-api" target="_blank">Azure Active Directory Graph API</a>
* <a href="https://jwt.io/" target="_blank">JWT.IO allows you to decode, verify and generate JWT</a>