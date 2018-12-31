---
layout: post
title: Use Azure DevOps Pipeline to Publish a PowerShell Module to the PowerShell Gallery
categories: [PowerShell, Azure, DevOps]
tags: [PowerShell, Azure, DevOps]
comments: true
---

Recently I released a new PowerShell Module called <a href="https://www.powershellgallery.com/packages/PSJwt" target="_blank">PSJwt to the PowerShell Gallery</a>. This is a PowerShell Module for JWT (JSON Web Tokens). The PowerShell Module is using the <a href="https://github.com/jwt-dotnet/jwt" target="_blank">Jwt.Net library</a>. This library supports generating and decoding <a href="https://tools.ietf.org/html/rfc7519" target="_blank">JSON Web Tokens</a>.

Part of the code for this PowerShell Module are the build steps and Azure DevOps YAML Build Pipeline.

In this blog post I explain how to use an Azure DevOps YAML Build Pipeline to Publish your PowerShell Module to the PowerShell Gallery.

## Build Steps
For the Build Steps I used the <a href="https://www.powershellgallery.com/packages/InvokeBuild" target="_blank">Invoke-Build PowerShell Module</a> from <a href="https://twitter.com/romkuzmin" target="_blank">Roman Kuzmin.</a> 

Invoke-Build/InvokeBuild is a build and test automation tool which invokes tasks defined in PowerShell v2.0+ scripts. It is similar to <a href="https://github.com/psake/psake" target="_blank">psake</a> but according to Roman easier to use and more powerful. It is complete, bug free, well covered by tests.

### Install InvokeBuild Module
Run:
```powershell
Install-Module -Name InvokeBuild -scope CurrentUser
```

And when you use Visual Studio Code I recommend you to also install the following helper PowerShell scripts:
* Invoke-TaskFromVSCode.ps1 - <a href="https://github.com/nightroman/Invoke-Build/wiki/Invoke-Task-from-VSCode" target="_blank">invokes a task from a build script opened in VSCode</a>
* New-VSCodeTask.ps1 - <a href="https://github.com/nightroman/Invoke-Build/wiki/Generate-VSCode-Tasks" target="_blank">generates VSCode tasks bound to build script tasks</a>

```powershell
Install-Script -Name Invoke-TaskFromVSCode
Install-Script -Name New-VSCodeTask
```

For development of the PowerShell Module I created the following tasks in the <a href="https://github.com/stefanstranger/psjwt/blob/master/PSJwt.build.ps1" target="_blank">PSJwt.build.ps1</a> script:
1. Task to Update the PowerShell Module Help Files
2. Task to retrieve latest version of JWT Packages
3. Task to Update JWT Package if newer version is released
4. Task to Copy PowerShell Module files to output folder for release as Module
5. Task to run all Pester tests in folder .\tests
6. Task to update the Module Manifest file with info from the Changelog in Readme
7. Task to Publish Module to PowerShell Gallery
8. Task clean up Output folder

**Note:**
```markdown
Not all the task defined in the Build script will be used in the Azure DevOps Build Pipeline.
We'll be using the following tasks in the Build Pipeline:
* Clean
* Copy PowerShell Modules files to output folder
* Publish Module to PowerShell Gallery
```

### Publish to PowerShell Gallery Task
For Publishing the PowerShell Module we need to use the PowerShell Cmdlet **Publish-Module** from the PowerShellGet PowerShell Module.

PublishModule InvokeBuild Task code:

```powershell
task PublishModule -If ($Configuration -eq 'Production') {
    Try {
        # Build a splat containing the required details and make sure to Stop for errors which will trigger the catch
        $params = @{
            Path        = ('{0}\Output\PSJwt' -f $PSScriptRoot )
            NuGetApiKey = $env:psgallery
            ErrorAction = 'Stop'
        }
        Publish-Module @params
        Write-Output -InputObject ('PSJwt PowerShell Module version published to the PowerShell Gallery')
    }
    Catch {
        throw $_
    }
}
```

I added a parameter to the <a href="https://github.com/stefanstranger/psjwt/blob/master/PSJwt.build.ps1" target="_blank">PSJwt.build.ps1</a> script to manage when the PowerShell Module should be publised to the Gallery.

If you call the task as follows the module will be published to the PowerShell Gallery:
```powershell
Invoke-Build -Configuration 'Production' -Task PublishModule
```
The <a href="https://docs.microsoft.com/en-us/powershell/gallery/how-to/publishing-packages/publishing-a-package#powershell-gallery-account-and-api-key" target="_blank">PowerShell Gallery API Key</a> is stored as an Environment variable called PSGallery. On your local system you can create the environment variable as follows.

```powershell
[Environment]::SetEnvironmentVariable('PSGallery', '[enter here PowerShell Gallery API Key]', 'User'
```

On the Azure DevOps (Build) Agent we will use a variable (NuGetAPIKey) to configure the PowerShell Gallery API Key.

## Azure DevOps Build Pipeline

Azure Pipelines is a cloud service that you can use to automatically build and test your code project and make it available to other users. It works with just about any language or project type.


Azure Pipelines combines continuous integration (CI) and continuous delivery (CD) to constantly and consistently test and build your code and ship it to any target. 

Last year Microsoft introduced YAML builds to give you another option for defining and evolving your continuous integration builds as your code evolves. I defined my Azure DevOps Build Pipeline in a <a href="https://github.com/stefanstranger/psjwt/blob/master/azure-pipelines.yml" target="_blank">YAML file</a>, part of the code in my Github Repository.

During the Build I want to execute some of the tasks I defined in my Invoke-Build script **PSJwt.build.ps1** and that is why I need to do some pre-requisites tasks like installing PowerShell Modules (InvokeBuild) before I can call the Invoke-Build PowerShell Function.

Let's have a look at the azure-pipelines.yml file:
```yaml
# Docs: https://aka.ms/yaml
name: $(Build.DefinitionName)_$(Date:yyyyMMdd))
pr:
- master

queue:
  name: Hosted VS2017

steps:
- powershell: .\bootstrap.ps1
  displayName: 'Install pre-requisites'

- task: richardfennellBM.BM-VSTS-PesterRunner-Task.Pester-Task.Pester@8
  displayName: 'Pester Test Runner'
  inputs:
    scriptFolder: '$(System.DefaultWorkingDirectory)\tests\*'
    additionalModulePath: '$(Build.ArtifactStagingDirectory)'
    CodeCoverageFolder: '$(Build.ArtifactStagingDirectory)'
    resultsFile: '$(Common.TestResultsDirectory)\Test-$(Build.DefinitionName)_$(Build.BuildNumber).xml'
    CodeCoverageOutputFile: '$(Common.TestResultsDirectory)\Coverage-$(Build.DefinitionName)_$(Build.BuildNumber).xml'

- task: PublishTestResults@2
  displayName: 'Publish Test Results'
  condition: always()
  inputs:
    testRunner: NUnit
    searchFolder: '$(Common.TestResultsDirectory)'

- task: PublishCodeCoverageResults@1
  displayName: 'Publish code coverage'
  inputs:
    summaryFileLocation: '$(Common.TestResultsDirectory)\Coverage-$(Build.DefinitionName)_$(Build.BuildNumber).xml'

- powershell: Invoke-Build -Configuration 'Production' -Task Clean, CopyModuleFiles, PublishModule
  displayName: 'Publish PowerShell Module'
  env:
    psgallery: $(NugetAPIKey)

- task: PublishBuildArtifacts@1
  displayName: 'Publish Artifact: Module'
  inputs:
    ArtifactName: Module
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
```
The following code defines the type of (Build) Agent where the build taks will be run on. In this case a Hosted VS2017 Agent.
```yaml
queue:
  name: Hosted VS2017
```
The next task in the Build will use the PowerShell Task to run the pre-requisites PowerShell script (<a href="https://github.com/stefanstranger/psjwt/blob/master/bootstrap.ps1" target="_blank">bootstrap.ps1)</a> which installs the following PowerShell modules, InvokeBuild, Pester and PlatyPS.

I found this bootstrap script solution on <a href="https://twitter.com/Jaykul" target="_blank">Joel Bennett's</a> <a href="https://github.com/Jaykul/PTUI" target="_blank">PTUI Github Repository</a>.

```yaml
- powershell: .\bootstrap.ps1
  displayName: 'Install pre-requisites'
```

The nexts tasks will run the Pester Tests using <a href="https://marketplace.visualstudio.com/items?itemName=richardfennellBM.BM-VSTS-PesterRunner-Task" target="_blank">Richard Fennell's Pester Test Runner Build Task</a>. This time I'm not using the Test task from my own build script, but you could also use the Test Task from my Build script if you wanted. I choose Richard's Task because it creates a results file which is used in the next task to publish the test results.

```yaml
- task: richardfennellBM.BM-VSTS-PesterRunner-Task.Pester-Task.Pester@8
  displayName: 'Pester Test Runner'
  inputs:
    scriptFolder: '$(System.DefaultWorkingDirectory)\tests\*'
    additionalModulePath: '$(Build.ArtifactStagingDirectory)'
    CodeCoverageFolder: '$(Build.ArtifactStagingDirectory)'
    resultsFile: '$(Common.TestResultsDirectory)\Test-$(Build.DefinitionName)_$(Build.BuildNumber).xml'
    CodeCoverageOutputFile: '$(Common.TestResultsDirectory)\Coverage-$(Build.DefinitionName)_$(Build.BuildNumber).xml'

- task: PublishTestResults@2
  displayName: 'Publish Test Results'
  condition: always()
  inputs:
    testRunner: NUnit
    searchFolder: '$(Common.TestResultsDirectory)'

- task: PublishCodeCoverageResults@1
  displayName: 'Publish code coverage'
  inputs:
    summaryFileLocation: '$(Common.TestResultsDirectory)\Coverage-$(Build.DefinitionName)_$(Build.BuildNumber).xml'
```

With the last task the Invoke-Build script (PSJwt.build.ps1) is run on the Build Agent. The tasks Clean, CopyModuleFiles and PublishMode are executed. 

```yaml
- powershell: Invoke-Build -Configuration 'Production' -Task Clean, CopyModuleFiles, PublishModule
  displayName: 'Publish PowerShell Module'
  env:
    psgallery: $(NugetAPIKey)
```

Note:
```markdown
You also need to define a Azure DevOps variable which contains your PowerShell Gallery NugetAPIKey value.
```

![Azure DevOps Variable](/assets/2018-12-30_16-57-45.png)

The NugetAPIKey variable will be configured as an environment variable which will be used during the PublishModule Build Task.

## Build Trigger
I want to trigger the Build and the Publish PowerShell Module task only for the Master Branch and when the PowerShell Module Manifest file is commited to the Github Repository.

![Azure DevOps Variable](/assets/2018-12-30_17-19-41.png)

Above configuration triggers the Build when a new Module Manifest (PSJwt.psd1) commit is made. All other file commits don't trigger a new build.

![PowerShell Gallery Result](/assets/2018-12-31_12-13-06.png)

Hope this inspired you to use Azure DevOps YAML Pipelines to publish your PowerShell Modules to the PowerShell Gallery.

**References:**

* <a href="https://www.powershellgallery.com/packages/PSJwt" target="_blank">PSJwt PowerShell Module on PowerShel Gallery</a>
* <a href="https://github.com/stefanstranger/PSJwt" target="_blank">Github Repository for PSJwt</a>
* <a href="https://www.powershellgallery.com/packages/InvokeBuild" target="_blank">InvokeBuild PowerShell Module on PowerShell Gallery</a>
* <a href="https://github.com/nightroman/Invoke-Build" target="_blank">Github Repository for Invoke-Build PowerShell Module</a>
* <a href="https://docs.microsoft.com/en-us/azure/devops/pipelines/get-started-yaml?view=vsts" target="_blank">Create your first pipeline</a>
* <a href="https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=vsts&tabs=schema" target="_blank">YAML schema reference</a>
* <a href="https://marketplace.visualstudio.com/items?itemName=richardfennellBM.BM-VSTS-PesterRunner-Task" target="_blank">Pester Test Runner Build Task</a>
* <a href="https://github.com/psake/psake" target="_blank">PSake PowerShell Module on Github</a>

