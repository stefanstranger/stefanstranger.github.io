---
layout: post
title: Everything you wanted to know about PowerShell's Module Path.
categories: [PowerShell, Scripts]
tags: [PowerShell, Scripts]
comments: true
---
While working on some PowerShell scripts which are using AzureRM PowerShell modules for an Azure DevOps Extension I had some trouble using the correct AzureRM PowerShell module version on the Hosted Agent.

By default version <a href="https://github.com/Microsoft/azure-pipelines-image-generation/blob/master/images/win/Vs2017-Server2016-Readme.md#azureazurerm-powershell-modules" target="_blank">2.1.0</a> of the PowerShell AzureRM version is being used on the Visual Studio 2017 on Windows Server 2016 (Hosted VS2017) Agent. But more AzureRM PowerShell versions are available, but not by default accessible via the PSModulePath environment variable.

It turned out that the PSModulePath determines which versions of a PowerShell Module are being used. The location of the PowerShell module must be in the PSModulePath environment variable, and the order of the paths in the PSModulePath determines which version is being used.

## PSModulePath
The PSModulePath environment variable stores the paths to the locations of the modules that are installed on disk. PowerShell uses this variable to locate modules when the user does not specify the full path to a module. The paths in this variable are searched in the order in which they appear.

If I look at my current PSModulePath environment variable on my Windows PowerShell host I see the following:

![](/assets//11112018-01.png)

When I retrieve the PSModulePath environment variable on PowerShell Core running on a Docker container with Ubuntu 16.04.5 I have the following paths in my PSModulePath:

![](/assets//11112018-02.png)

![](/assets//11112018-03.png)

## Modifying the PSModulePath
There are multiple ways to modify the PSModulePath variable.

* To add a temporary value that is available only for the current session, run the following command at the command line:
    ```powershell
    $env:PSModulePath = $env:PSModulePath + ";c:\ModulePath"
    ```
    For PowerShell Core on Linux run the following:
    ```powershell
    $env:PSModulePath = $env:PSModulePath + ":/ModulePath"
    ```
    
    Screenshot of updated PSModulePath environment variable for PowerShell Core running on Ubuntu Docker container image.
    
    ![](/assets//11112018-04.png)

* To add a persistent value that is available whenever a session is opened, add the following command to a **Windows PowerShell profile**:
    ```powershell
    $env:PSModulePath = $env:PSModulePath + ";c:\ModulePath"
    ```
    For more information about profiles, see **<a href="https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_profiles" target="_blank">about_Profiles</a>** in the Microsoft TechNet library.

* To add a persistent variable to the registry, create a new user environment variable called PSModulePath using the Environment Variables Editor in the **System Properties** dialog box.
* To add a persistent variable by using a script, use the **SetEnvironmentVariable** method on the Environment class. For example, the following script adds the "C:\Program Files\Fabrikam\Module path to the value of the PSModulePath environment variable for the computer. To add the path to the user PSModulePath environment variable, set the target to "User".
    ```powershell
    $CurrentValue = [Environment]::GetEnvironmentVariable("PSModulePath", "Machine")
    [Environment]::SetEnvironmentVariable("PSModulePath", $CurrentValue + ";C:\Program Files\Fabrikam\Modules", "Machine")
    ```

## PSModulePath in Action
On my Windows 10 development system I've the following PSModulePath configured:

![](/assets//11112018-01.png)

Let's first see which version of the AzureRM.Resources Module will be used when I try to use the Get-AzureRMResourceGroup Cmdlet.

```powershell
Get-Command -Name Get-AzureRMResourceGroup
```

![](/assets//11112018-05.png)

This shows that Version 6.5.0 of the AzureRM.Resources PowerShell module is being used.

This module is being located via the PSModulePath environment variable in folder 
C:\Users\\[username]\Documents\WindowsPowerShell\Modules\AzureRM.Resources.

![](/assets//11112018-06.png)

This is also the first path where the AzureRM.Resources PowerShell Module is located.

![](/assets//11112018-07.png)

**What would happen if we changed the PSModulePath to have a different path as first location to contain the AzureRM.Resources Module?**

**Remark:**

*Use a 'clean' PowerShell host to avoid already having loaded a PowerShell module in the session.*

```powershell
# Adding new path to the beginning of the PSModulePath!
$env:PSModulePath = "C:\Modules\azurerm_5.1.1;" + $env:PSModulePath
```

![](/assets//11112018-08.png)

Check now which PowerShell AzureRM.Resources module version will be loaded when retrieving the Get-AzureRMResourceGroup cmdlet.

```powershell
Get-Command -Name Get-AzureRMResourceGroup
```

![](/assets//11112018-09.png)


Now the first PowerShell AzureRM.Resources module being found in the PSModulePath variable is version 5.1.1 which is located in the path C:\Modules\azurerm_5.1.1\5.1.1\AzureRM.Resources.

![](/assets//11112018-10.png)

We have now seen that the PSModulePath looks for the **highest/latest** PowerShell Module version in the **order** of the configured paths in the PSModulePath environment variable.

## Remove locations from PSModulePath

With the following method you can reset the PSModulePath to the default setting again:
```powershell
$CurrentValue = [Environment]::GetEnvironmentVariable("PSModulePath", "Machine")
$UserPSModuleLocation = "$HOME\Documents\WindowsPowerShell\Modules"
$Env:PSModulePath = $UserModuleLocation + ";" + $CurrentValue
```

**Conclusion:**

If you want to use the PSModulePath for autoloading PowerShell modules make sure your desired PowerShell Module version path is in the first order of the PSModulePath variable.

## $PSModuleAutoloadingPreference
Only modules that are stored in the location specified by the PSModulePath environment variable are automatically imported. Modules in other locations must be imported by running the Import-Module cmdlet.

The **$PSModuleAutoloadingPreference** enables and disables automatic importing of modules in the session. "All" is the default. Regardless of the value of this variable, you can use the Import-Module cmdlet to import a module.
Valid values are:
* All: Modules are imported automatically on first-use. To import a module, get (Get-Command) or use any command in the module.
* ModuleQualified: Modules are imported automatically only when a user uses the module-qualified name of a command in the module. For example, if the user types "MyModule\MyCommand", PowerShell imports the MyModule module.
* None: Automatic importing of modules is disabled in the session. To import a module, use the Import-Module cmdlet.
For more information about automatic importing of modules, see <a href="https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_modules?view=powershell-6" target="_blank">about_Modules</a>.

You can disable the automatic loading of module by using the following command:

```powershell
$PSModuleAutoloadingPreference = 'none'
```

**Note**

I do not recommend you make this change except in very specific situations and for very specific reasons. The number of Windows PowerShell cmdlets and functions in Windows Server 2018 and Windows 10 would make knowing which module a particular command resided in extremely difficult; with this change in place, you have to specifically load the module prior to using any commands.

Let's see what happens if we disable auto module loading when retrieving the Get-AzureRMResourceGroup cmdlet.

**Remark:**

*Use a 'clean' PowerShell host to avoid already having loaded a PowerShell module in the session.*

![](/assets//11112018-11.png)

The Command is not found. We need to explictly use Import-Module to be able to find and use the Get-AzureRmResourceGroup cmdlet.

```powershell
Import-Module C:\Modules\azurerm_5.1.1\5.1.1\AzureRM.Resources -RequiredVersion 5.1.1
```

![](/assets//11112018-12.png)

After explictly importing a specific PowerShell Module and version we are able to use the cmdlet we want.

**Conclusions**

* If you have auto module loading enabled, make sure you have the required PowerShell module on the **first position** of your **PSModulePath** Environment variable, when having multiple locations for the same PowerShell module.
* PowerShell uses the **latest version** of a Module when it finds multiple versions of the PowerShell module in the PSModulePath environment variable.




**References:**
* <a href="https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=vsts&tabs=yaml" target="_blank">Microsoft-hosted agents</a>
* <a href="https://github.com/Microsoft/azure-pipelines-image-generation/blob/master/images/win/Vs2017-Server2016-Readme.md#azureazurerm-powershell-modules" target="_blank">PowerShell AzureRM version on Hosted VS2017 Agent</a>
* <a href="https://docs.microsoft.com/en-us/powershell/developer/module/modifying-the-psmodulepath-installation-path" target="_blank">Modifying the PSModulePath Installation Path</a>
* <a href="https://blogs.technet.microsoft.com/heyscriptingguy/2013/02/20/powertip-turn-off-powershell-module-autoload/" target="_blank">PowerTip: Turn Off PowerShell Module Autoload</a>