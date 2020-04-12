---
layout: post
title: Creating Azure DevOps WIKI Pages from within a pipeline - part 1
categories: [CI/CD, WIKI]
tags: [CI/CD, WIKI, Markdown]
comments: true
---

For a project I was investigating how I could dynamically create Markdown files. I searched the internet and I found the following PowerShell Module from <a href="https://www.linkedin.com/in/bernie-white" target="_blank">Bernie White</a> a Premier Field Engineer from Microsoft.

## PSDocs

PSDocs is a PowerShell module with commands to generate markdown from objects using PowerShell syntax.

According to it's disclaimer the project is a **proof-of-concept** but for me it worked great and in these blog post series I'll explain how you can use this in your Azure DevOps Pipelines to create Markdown Wiki documentation. In part one I start with the basics.

### Installation of PSDocs
To install the PSDocs PowerShell module run:

```PowerShell
Install-Module -Name PSDocs -Scope CurrentUser
```

### Getting started

To create a Markdown content object stored running process in a Markdown table you can use the following example.

```PowerShell
#region Create WIKI MarkDown Content
Document 'Create-MarkdownTable' {

    'Demo for creating WIKI Pages'

    Section 'Process Table' {
        $InputObject | Table
    } 
}

# Generate markdown for the inline document
# Markdown options to use Edge Pipelines for the Markdown tables.
$options = New-PSDocumentOption -Option @{ 'Markdown.UseEdgePipes' = 'Always'; 'Markdown.ColumnPadding' = 'None' };
$null = [PSDocs.Configuration.PSDocumentOption]$Options

# $InputObject is set by using the -InputObject parameter of Invoke-PSDocument or inline functions. The value of the pipeline object currently being processed.
$InputObject = Get-Service | Select-Object -First 5

# Using the PassThru parameter of Create-MarkdownTable function allows for storing the output in a variable.
$Content = Create-MarkdownTable -InputObject $InputObject -Option $Options -PassThru
#endregion
```

![Example PSDocs animated gif](/assets/PSDocs1.gif)

In Markdown this will look like this.

![MarkDown result](/assets/2020-04-12_12-49-21.png)


Now we know how to create a Markdown document we need to learn to who create an Azure DevOps Wiki page. 

## Azure DevOps REST API - Pages

According to the [documentation](https://docs.microsoft.com/en-US/rest/api/azure/devops/wiki/pages?view=azure-devops-rest-5.0) Wiki pages are Markdown files that are stored in a Git repository in the backend.

The following web requests creates or edits a wiki page.

```http
PUT https://dev.azure.com/{organization}/{project}/_apis/wiki/wikis/{wikiIdentifier}/pages?path={path}&api-version=5.0
```

### Wiki Identifier

One of the needed properties to create the wiki page is the wikiIdentifier. This is the Wiki id or name of the wiki.

We can find the wiki name by going to the Azure DevOps Project and selecting Wiki in the menu.

![Name of wiki](/assets/2020-04-12_14-26-38.png)

### PowerShell script to create an Azure DevOps Wiki page

With the following script you can create a Wiki page in Azure DevOps using the PSDocs PowerShell module and the Azure DevOps REST API.

```PowerShell
<#
    PowerShell script to create Azure DevOps WIKI Markdown Documentation

    https://docs.microsoft.com/en-US/rest/api/azure/devops/wiki/pages/create%20or%20update?view=azure-devops-rest-5.0#examples
    https://medium.com/digikare/create-automatic-release-notes-on-azuredevops-f235376ec533

    Requirements:
    - PSDocs PowerShell Module (Install-Module -Name PSDocs)
#>

#region variables
$OrganizationName = '[enter Azure DevOps Organization Name]'
$ProjectName = '[enter Azure DevOps Project Name]'
$PAT = '[enter Personal Access Token]'
$WikiName = '[enter wiki Name]'
#endregion

#region Create WIKI MarkDown Content
Document 'Create-MarkdownTable' {

    'Demo for creating WIKI Pages'

    Section 'Process Table' {
        $InputObject | Table
    } 
}

# Generate markdown for the inline document
$options = New-PSDocumentOption -Option @{ 'Markdown.UseEdgePipes' = 'Always'; 'Markdown.ColumnPadding' = 'None' };
$null = [PSDocs.Configuration.PSDocumentOption]$Options
$InputObject = Get-Service | Select-Object -First 5
$Content = Create-MarkdownTable -InputObject $InputObject -Option $Options -PassThru
#endregion


#region Create WIKI page
$uri = ('https://dev.azure.com/{0}/{1}/_apis/wiki/wikis/{2}/pages?path={3}&api-version=5.0' -f $OrganizationName, $ProjectName, $WikiName, $WikiName)

$Header = @{
    'Authorization' = 'Basic ' + [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(":$($PAT)")) 
}

$params = @{
    'Uri'         = $uri
    'Headers'     = $Header
    'Method'      = 'Put'
    'ContentType' = 'application/json; charset=utf-8'
    'body'        = @{content = $content; } | ConvertTo-Json
}

Invoke-RestMethod @params
#endregion
```

![wiki page screenshot](/assets/2020-04-12_14-34-24.png)

If you want to delete the wiki page using the REST API you can use the following command:

```PowerShell
#region variables
$OrganizationName = '[enter Azure DevOps Organization Name]'
$ProjectName = '[enter Azure DevOps Project Name]'
$PAT = '[enter Personal Access Token]'
$WikiName = '[enter wiki Name]'
#endregion

#region delete page
$uri = ('https://dev.azure.com/{0}/{1}/_apis/wiki/wikis/{2}/pages?path={3}&api-version=5.0' -f $OrganizationName, $ProjectName, $WikiName, $Wikipage)
$Header = @{
    'Authorization' = 'Basic ' + [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(":$($PAT)")) 
}

$params = @{
    'Uri'         = $uri
    'Headers'     = $Header
    'Method'      = 'Delete'
    'ContentType' = 'application/json; charset=utf-8'
}

Invoke-RestMethod @params
#endregion
```

In the next part of this blog post I want to share how you can use above knowledge to create a wiki from within an Azure DevOps Pipeline.

Hope you enjoyed this blog post.

**References**

- [Generate markdown from PowerShell](https://github.com/BernieWhite/PSDocs)

- [Azure DevOps REST API documentation for Wiki pages](https://docs.microsoft.com/en-US/rest/api/azure/devops/wiki/pages?view=azure-devops-rest-5.)

- [Azure DevOps REST API for Pages](https://docs.microsoft.com/en-US/rest/api/azure/devops/wiki/pages?view=azure-devops-rest-5.0)
