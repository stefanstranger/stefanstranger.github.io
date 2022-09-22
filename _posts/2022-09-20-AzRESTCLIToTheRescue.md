---
layout: post
title: Az REST Cli to the Rescue
categories: [Az CLI, REST API, Automation]
tags: [Az CLI, REST API, Automation]
comments: true
---

- [Introduction](#introduction)
- [REST API process flow](#rest-api-process-flow)
- [Example Azure Resource Manager REST API call](#example-azure-resource-manager-rest-api-call)
  - [Steps 1 & 2 Login to the Authorization Server and get Token](#steps-1--2-login-to-the-authorization-server-and-get-token)
  - [Steps 2 & 3 Use Token to get overview of Azure Subscroptions](#steps-2--3-use-token-to-get-overview-of-azure-subscroptions)
- [Using the Az CLI REST Command](#using-the-az-cli-rest-command)
- [References:](#references)

# Introduction
Within a Logic App I needed to run a Log Analytics search query and I had trouble understanding how this REST API call should be created.

First I looked at the existing documentation <a href="https://learn.microsoft.com/en-us/rest/api/loganalytics/dataaccess/query/get?tabs=HTTP" target="_blank">here</a>.

According to the documentation the following Authentication/Authorization is required.

![](/assets/20-09-2022-01.png)

But I was missing information about the scope required for the access token that could be used for the REST API call.

In this blog post I'm explaining how you can use the Az CLI REST command to figure out how to call the Log Analytics REST API.

From the documentation we know that you can communicate with the Azure Monitor Log Analytics API using this endpoint: https://api.loganalytics.io. To access the API, you must authenticate through Azure Active Directory (Azure AD).

Before I explain how we can use the Az CLI REST command let's first go to a standard REST API flow to get all the Azure Subscriptions.

# REST API process flow

![](/assets/20-09-2022-02.png)

# Example Azure Resource Manager REST API call

Looking at above REST API process flow, how would that look like for getting the Azure Subscriptions from the Azure Resource Manager using PowerShell?

## Steps 1 & 2 Login to the Authorization Server and get Token

```PowerShell
#region variables
# Set well-known client ID for Azure PowerShell
$clientId = '1950a258-227b-4e31-a9cf-717495945fc2'
$tenantId     = '{tenantid}'
#endregion
 

#region Login and Get Access Token
$authUrl = ('https://login.microsoftonline.com/{0}/oauth2/v2.0/token' -f $tenantid) 
$scope = 'https://management.core.windows.net//.default'

$body = @{
        'client_id' = $clientId
        'grant_type' = 'password'
        'username' = $userName
        'password' = $password
        'scope' = $scope
}

$params = @{
    ContentType = 'application/x-www-form-urlencoded'
    Headers = @{'accept'='application/json'}
    Body = $body
    Method = 'Post'
    URI = $authUrl
}

Invoke-RestMethod @params -OutVariable AccessToken
#endregion
```

## Steps 2 & 3 Use Token to get overview of Azure Subscroptions

```PowerShell

#region Get Azure Subscriptions
$subscriptionURI = 'https://management.azure.com/subscriptions?api-version=2020-01-01'

$params = @{
    ContentType = 'application/json'
    Headers = @{
    'authorization'="Bearer $($AccessToken.access_token)"
    }
    Method = 'Get'
    URI = $subscriptionURI
}

Invoke-RestMethod @params
#endregion
```

Screenshot result of Azure Subscriptions

![](/assets/20-09-2022-03.png)


# Using the Az CLI REST Command

One of the missing parts of being able to call the Log Analytics REST API was documentation on the Authentication and Authorization flow.

In this specific case I know the url that I want to call.

| HTTP | 
|----------------|
| GET https://api.loganalytics.io/v1/workspaces/{workspaceId}/query?query={query}| 

But I don't have the information for getting an Access Token except that I need authenticate through Azure Active Directory (Azure AD). From the previous example we have learned that we need to know the scope to get the correct Access Token.

Let's start with using the Az CLI and start to with the login.

```cmd
az login
```

Screenshot from az login command

![](/assets/20-09-2022-04.png)

According to the documentation of az rest, this command automatically authenticates using the logged-in credential: If Authorization header is not set, it attaches header Authorization: Bearer <token>, where <token> is retrieved from AAD. The target resource of the token is derived from --url if --url starts with an endpoint from az cloud show --query endpoints.

The az cli has a --debug switch which can help to get all the info we are looking for. To be able to show the debug stream we open a command prompt (not a PowerShell) and run the following:


```cmd
 az rest --method get --url "https://api.loganalytics.io/v1/workspaces/{workspace}/query?query={query]" --debug
```

The debug info provides us with all the information we are looking for.

![](/assets/20-09-2022-05.png)

Now we use this for below PowerShell code to call the Log Analytics REST API.

```PowerShell
#region variables
# Set well-known client ID for Azure PowerShell
$clientId = '1950a258-227b-4e31-a9cf-717495945fc2'
$tenantId     = '{tenantid}'
#endregion
 

#region Get Access Token
$authUrl = ('https://login.microsoftonline.com/{0}/oauth2/v2.0/token' -f $tenantid) 
$scope = 'https://api.loganalytics.io/.default'

$body = @{
        'client_id' = $clientId
        'grant_type' = 'password'
        'username' = $userName
        'password' = $password
        'scope' = $scope
}

$params = @{
    ContentType = 'application/x-www-form-urlencoded'
    Headers = @{'accept'='application/json'}
    Body = $body
    Method = 'Post'
    URI = $authUrl
}

Invoke-RestMethod @params -OutVariable AccessToken
#endregion

#region Call Log Analytics REST API
$uri= 'https://api.loganalytics.io/v1/workspaces/{workspaceid}/query?query={query}'

$params = @{
    ContentType = 'application/json'
    Headers = @{
    'authorization'="Bearer $($AccessToken.access_token)"
    }
    Method = 'GET'
    URI = $uri
}

Invoke-RestMethod @params | Select-Object -ExpandProperty tables
#endregiont

```

Screenshot of result.

![](/assets/20-09-2022-06.ng.png)

Hope this is of interest and I want to thank [Jos Koelewijn](https://twitter.com/Jawz_84) for pointing out to use the Az CLI rest command. 

# References:

- [What is Authentication?](https://infotecinformation.wordpress.com/2021/08/06/oauth2/)

- [Using the Azure ARM REST API – Get Subscription Information](https://stefanstranger.github.io/2016/10/29/using-the-azure-arm-rest-api/)

- [Log Analytics REST API Reference](https://learn.microsoft.com/en-us/rest/api/loganalytics/)

- [Az CLI REST Command documentation](https://learn.microsoft.com/en-us/cli/azure/reference-index?view=azure-cli-latest#az-rest)
