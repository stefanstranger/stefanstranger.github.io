---
layout: post
title: Automatically generating PowerShell cmdlets to manage or access a RESTful Web Service?
categories: [REST API, PowerShell]
tags: [REST API, PowerShell]
---
# Automatically generating PowerShell cmdlets to manage or access a RESTful Web Service?

This week there was a conversation on Twitter about creating a WorldCup PowerShell Module using PSSwagger.

<a href="https://twitter.com/ehrnst/status/1007693837643460610" target="_blank">![](/assets/twitter.png)</a>

I never let a good opportunity slip to learn something new, so I started to investigate the PowerShell PSSwagger module first.

But first what is Swagger?

**Swagger**

Swagger is a powerful yet easy-to-use suite of API developer tools for teams and individuals, enabling development across the entire API lifecycle, from design and documentation, to test and deployment.

Swagger consists of a mix of open source, free and commercially available tools that allow anyone, from technical engineers to street smart product managers to build amazing APIs that everyone loves.

Swagger is built by SmartBear Software, the leader in software quality tools for teams. SmartBear is behind some of the biggest names in the software space, including Swagger, SoapUI and QAComplete.

[from https://swagger.io/about/]

**PSSwagger**

PSSwagger is a PowerShell Cmdlet generator for OpenAPI based web services. 

Some of the benefits of PSSwagger are:
* One module that works cross-platform – generated modules supports both PowerShell Core and Windows PowerShell 5.1.
* Consistent user experience – the generated PowerShell commands automatically follow the PowerShell cmdlet guidelines and best practices.
* Get additional features for free – paging and asynchronous execution of long running operations.
* Web Service owners can minimize the effort to author PowerShell commands to manage or access their web service.
* Ability to load multiple different versions of a generated module into the same process.

[from: PowerShell Team Blog post]


**Swagger introduction**

Before we can start to automatically generate PowerShell cmdlets from an OpenAPI (f.k.a Swagger) specification, we first need to learn a bit more about Swagger.

A good starting place is a the <a href="http://idratherbewriting.com/learnapidoc/pubapis_swagger.html" target="_blank">Swagger UI tutorial</a> on the Documenting APIs website.

After reading this tutorial we can start playing with Swagger using the <a href="http://petstore.swagger.io/" target="_blank">Swagger Petstore example</a>. In the Petstore example, the site is generated using Swagger UI.

Now we have some more information about Swagger, it's time to generate the PowerShell cmdlets for the Swagger Petstore Example.

**Installation PSSwagger**
1. Install PSSwager from PowerShell Gallery
```powershell
Install-Module -Name PSSwagger -Scope CurrentUser
```
2. Install AutoRest
* Install node.js (More info can be found here: https://github.com/Azure/autorest/blob/master/docs/developer/workstation.md#nodejs)
* Install AutoRest using npm
```javascript
npm install -g autorest
```
3. Ensure AutoRest is installed and available in $env:PATH
```powershell
$env:path += ";$env:APPDATA\npm"
Get-Command -Name AutoRest
```
4. Ensure CSC.exe is installed and available in $env:PATH
```powershell
Install-Package -Name Microsoft.Net.Compilers -Source https://www.nuget.org/api/v2 -Scope CurrentUser

$package = Get-Package -Name Microsoft.Net.Compilers
$env:path += ";$(Split-Path $package.Source -Parent)\tools"
Get-Command -Name CSC.exe
```
Verify if AutoRest is working by running autorest from the commandline.

I initially got the following error message:

![](/assets/autoresterror.png)

Running
```powershell
autorest --reset
```
solved this.

We should now be ready to automatically create PowerShell Cmdlets for the Swagger Petstore.

The following code created the Swagger.PetStore PowerShell Module.

```powershell
Import-Module psswagger

$TargetPath = 'C:\Temp\petstore'

$param = @{
    Path                    = $TargetPath
    UseAzureCsharpGenerator = $false
    IncludeCoreFxAssembly   = $false
}


#region create PowerShell Module for Swagger PetStore
$param['SpecificationUri'] = 'https://petstore.swagger.io/v2/swagger.json'
$param['Name'] = 'Swagger.PetStore'

New-PSSwaggerModule @param
#endregion
```
**Using the Swagger.PetStore PowerShell Module**

Here are some examples for using the Swagger.PetStore PowerShell Module.

```powershell
Import-Module C:\temp\petstore\Swagger.PetStore\0.0.1\Swagger.PetStore.psd1

#region retrieve Commands from Swagger.PetStore Module
Get-Command -module Swagger.PetStore
#endregion

#region Create Swagger.PetStore Object
New-CategoryObject -Id 1 -Name 'Dogs' -OutVariable Category
New-TagObject -Id 1 -Name 'Dog' -OutVariable Tags

New-PetObject -Id 9834132 -Category $Category[0] -Name 'MyDog' -Tags $Tags[0] -PhotoUrls 'www.contoso.com' -Status 'Available' -OutVariable Pet
#endregion

#region Add Pet object to Swagger.Petstore REST API
Add-Pet -Body $Pet[0]
#endregion

#region Get Pet Object from Swagger.PetStore REST API
Get-PetById -PetId 9834132 -APIKey 'dummy'
#endregion
```
We can now see our PetStore Object using PowerShell.

![](/assets/PetStoreResults.png)

Have fun exploring Swagger REST API using the PSSwagger module!


**References:**
* <a href="http://idratherbewriting.com/learnapidoc/pubapis_swagger.html" target="_blank">Swagger UI tutorial</a>
* <a href="https://swagger.io/" target="_blank">Swagger web site</a>
* <a href="https://blogs.msdn.microsoft.com/powershell/2017/08/15/psswagger-automatically-generate-powershell-cmdlets-from-openapi-f-k-a-swagger-specification/" target="_blank">PowerShell Team Blog post: PSSwagger – Automatically generate PowerShell cmdlets from OpenAPI (f.k.a Swagger) specification</a>
* <a href="http://petstore.swagger.io/" target="_blank">Swagger PetStore Example</a>
* <a href="https://github.com/PowerShell/PSSwagger" target="_blank">PSSwagger Github</a>
* <a href="https://github.com/Azure/AutoRest" target="_blank">AutoRest on Github</a>