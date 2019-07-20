---
layout: post
title: Passing variables from stage to stage in Azure DevOps Release Pipelines
categories: [Azure DevOps, CI/CD]
tags: [Azure DevOps, CI/CD]
comments: true
---

While working on an Azure DevOps Release Pipeline I wanted to pass a variable from one Stage to another Stage and it turned out this was not possible without some extra effort.

After some searches on the Internet I found that <a href="http://donovanbrown.com/post/Passing-variables-from-stage-to-stage-in-Azure-DevOps-release" target="_blank">Donovan Brown</a> also wrote a blog post on this topic but I was not able to get this working.

**Update 07-30-2019:** 

It turned out that it didn't work due to a permission issue. Check his original blog post for more information.

# Variable Scopes

Azure DevOps has various scopes where you can define your custom variables.

* Share values across all of the definitions in a project by using <a href="https://docs.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups?view=azure-devops" target="_blank">**variable groups**</a>.   
Choose a variable group when you need to use the same values across all the definitions, stages, and tasks in a project, and you want to be able to change the values in a single place. You define and manage variable groups in the Library tab.

* Share values across all of the stages by using **release pipeline variables**.  
Choose a release pipeline variable when you need to use the same value across all the stages and tasks in the release pipeline, and you want to be able to change the value in a single place. You define and manage these variables in the Variables tab in a release pipeline. In the Pipeline Variables page, open the Scope drop-down list and select "Release". By default, when you add a variable, it is set to Release scope.

* Share values across all of the tasks within one specific stage by using **stage variables**.
Use a stage-level variable for values that vary from stage to stage (and are the same for all the tasks in an stage). You define and manage these variables in the Variables tab of a release pipeline. In the Pipeline Variables page, open the Scope drop-down list and select the required stage. When you add a variable, set the Scope to the appropriate environment.

## Release Pipeline variable
If we want to share variables across multiple stages we need to create or set a Release Pipeline variable.

## Defining and modifying variables in a script
To define or modify a variable from a script, use the task.setvariable logging command. **Note that the updated variable value is scoped to the job being executed, and does not flow across jobs or stage**.

```PowerShell
Write-Host "##vso[task.setvariable variable=FirstName]Stefan"
```

And this is exactly what we want to do in this blog post.

So how can create a variable in one Stage and pass that variable to another Stage in your Release Pipeline?

# Update Release Definition

The solution is updating the Release Definition for the Release Pipeline variable in the Stage where the variable is set.

To update the Release Definition we are using the <a href="https://docs.microsoft.com/en-us/rest/api/azure/devops/release/definitions/update?view=azure-devops-rest-5.0" target="_blank">Azure DevOps REST API</a>. 

## Release Permissions
To allow for updating the Release Definition during the Release you need to configure the Release Permission **Manage releases** for the **Project Collection Build Service**.

![](/assets/2019-06-26-01.png)

## Allow scripts to access the OAuth token
Because we are going to update the Release Definition and Release variable in the first Stage we need to enable the Allow scripts to access the OAuth token.

![](/assets/2019-06-26-02.png)

# Example Release Pipeline

Let's start with the creation of new Azure DevOps Release Pipeline and start with an Empty job.

![](/assets/2019-06-26-03.png)

Next create an empty Pipeline variable for the Release scope.

![](/assets/2019-06-26-04.png)

Configure Allow scripts to access the OAuth token on the Agent job in Stage 1.

Add an (inline) PowerShell script task to create a variable in Stage 1.

Example code:
```PowerShell
$Random = @('Foo','Bar','Baz','Qux') | Get-Random

Write-Output -InputObject ('Variable myVar in Task 1, Stage 1 is: {0}' -f $Random)

Write-Output ('##vso[task.setvariable variable=myVar]{0}' -f $Random)
```
![](/assets/2019-06-26-05.png)

Now it's time to update the Release Definition and Release Variable (StageVar). Again create an (inline) PowerShell task in Stage 1.

Use the following code example as inline script.

```PowerShell
#region variables
$ReleaseVariableName = 'StageVar'
$releaseurl = ('{0}{1}/_apis/release/releases/{2}?api-version=5.0' -f $($env:SYSTEM_TEAMFOUNDATIONSERVERURI), $($env:SYSTEM_TEAMPROJECTID), $($env:RELEASE_RELEASEID)  )
#endregion


#region Get Release Definition
Write-Host "URL: $releaseurl"
$Release = Invoke-RestMethod -Uri $releaseurl -Headers @{
    Authorization = "Bearer $env:SYSTEM_ACCESSTOKEN"
}
#endregion

#region Output current Release Pipeline
Write-Output ('Release Pipeline variables output: {0}' -f $($Release.variables | ConvertTo-Json -Depth 10))
#endregion


#region Update StageVar with new value
$release.variables.($ReleaseVariableName).value = "$(myVar)"
#endregion

#region update release pipeline
Write-Output ('Updating Release Definition')
$json = @($release) | ConvertTo-Json -Depth 99
Invoke-RestMethod -Uri $releaseurl -Method Put -Body $json -ContentType "application/json" -Headers @{Authorization = "Bearer $env:SYSTEM_ACCESSTOKEN" }
#endregion

#region Get updated Release Definition
Write-Output ('Get updated Release Definition')
Write-Host "URL: $releaseurl"
$Release = Invoke-RestMethod -Uri $releaseurl -Headers @{
    Authorization = "Bearer $env:SYSTEM_ACCESSTOKEN"
}
#endregion

#region Output Updated Release Pipeline
Write-Output ('Updated Release Pipeline variables output: {0}' -f $($Release.variables | ConvertTo-Json -Depth 10))
#endregion
```

Remark:
Make sure you use "$(myVar)" to retrieve the script value! Replace myVar with your variable name.

![](/assets/2019-06-26-06.png)

Create a new Stage 2 and verify if the myVar variable can be retrieved.

![](/assets/2019-06-26-07.png)

Again we can use an (inline) PowerShell task to retrieve the value of myVar via the Release Variable StageVar.

![](/assets/2019-06-26-08.png)

```PowerShell
Write-Output -InputObject ("myVar Variable in Task 1, Stage 2 is: {0}" -f "$(stageVar)")
```

Result:

Release variable set in Stage 1:
![](/assets/2019-06-26-09.png)

Retrieving variable in Stage 2:
![](/assets/2019-06-26-10.png)

Hope this will help you when passing Release Variable from one Stage to another Stage.

Thanks <a href="https://www.linkedin.com/in/ejderooij/" target="_blank">Erik de Rooij</a> for helping me with solving the code to update the Release Definition.


















**References:**
* <a href="http://donovanbrown.com/post/Passing-variables-from-stage-to-stage-in-Azure-DevOps-release" target="_blank">Blog Post: Passing variables from stage to stage in Azure DevOps release from Donovan Brown</a>
* <a href="https://docs.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops&tabs=yaml%2Cbatch" target="_blank">Azure DevOps Variables</a>
* <a href="https://docs.microsoft.com/en-us/azure/devops/pipelines/release/variables?view=azure-devops&tabs=batch" target="_blank">Release variables Documentation</a>
* <a href="https://docs.microsoft.com/en-us/rest/api/azure/devops/release/definitions/update?view=azure-devops-rest-5.0" target="_blank">Azure DevOps REST API for Updating Release Definition</a>.

