---
layout: post
title: Develop and run a timer triggered Azure Function locally
categories: [Azure]
tags: [Azure Functions, Azure, Development]
comments: true
---

While developing a poorman's IPAM solution using Azure Functions and a Storage Table I wanted to test a timer triggered Azure Function locally.

This blog post explains how I was able to develop the (PowerShell) timer triggered Azure Function.
- [Local develop environments](#local-develop-environments)
- [Create a new (PowerShell) Azure Function](#create-a-new-powershell-azure-function)
- [Start Azure Function locally](#start-azure-function-locally)
- [Install Azurite Storage Emulator](#install-azurite-storage-emulator)
- [Start Azurite Storage Emulator](#start-azurite-storage-emulator)
- [Restart Timer triggered Azure Function](#restart-timer-triggered-azure-function)
- [References](#references)

# Local develop environments

There are multiple ways to set up you local development environment. See the [Code and test Azure Functions locally](https://docs.microsoft.com/en-us/azure/azure-functions/functions-develop-local) documentation for more information.

First I installed the the Azure Functions Core Tools using Chocolatey but you can use whatever method you feel comfortable with.

```PowerShell
choco install azure-functions-core-tools-3
```

Current version of the Azure Function Core Tools on my development machine is 3.0.3160.

# Create a new (PowerShell) Azure Function

Again depending on the setup of your local develop environment there are multiple ways to create a new Azure Function but I used the commandline option using the Azure Function Core tools.

```PowerShell
func new --language powershell --template timertrigger --name timertrigger
Use the up/down arrow keys to select a worker runtime:powershell
Writing profile.ps1
Writing requirements.psd1
Writing .gitignore
Writing host.json
Writing local.settings.json
Writing C:\temp\azfuncblog\.vscode\extensions.json
Use the up/down arrow keys to select a language:Use the up/down arrow keys to select a template:timertrigger
Function name: [TimerTrigger] Writing C:\temp\azfuncblog\timertrigger\readme.md
Writing C:\temp\azfuncblog\timertrigger\run.ps1
Writing C:\temp\azfuncblog\timertrigger\function.json
The function "timertrigger" was created successfully from the "timertrigger" template.
```

# Start Azure Function locally

If we now try to run the just created Azure Function locally using func start --verbose we get the following error message.

![Starting Azure Function](/assets/06-01-2021-01.png)

# Install Azurite Storage Emulator

To fix above error we need to install an Azure Storage Emulutor like Azurite. I installed Azurite using NPM.

This installation method requires that you have Node.js version 8.0 or later installed. Node Package Manager (npm) is the package management tool included with every Node.js installation. After installing Node.js, execute the following npm command to install Azurite.

```PowerShell
npm install -g azurite
```

# Start Azurite Storage Emulator

After having installed the Azurite Storage Emulator we can start the Azurite with the following command.

```PowerShell
azurite
Azurite Blob service is starting at http://127.0.0.1:10000
Azurite Blob service is successfully listening at http://127.0.0.1:10000
Azurite Queue service is starting at http://127.0.0.1:10001
Azurite Queue service is successfully listening at http://127.0.0.1:10001
Azurite Table service is starting at http://127.0.0.1:10002
Azurite Table service is successfully listening at http://127.0.0.1:10002
```

# Restart Timer triggered Azure Function

We should now be able to run the timer triggered Azure Function and check if everything is working as expected.

![Animated gif](/assets/AzureFunctionTimerTrigger.gif)

Hope this helps developing your Azure Functions locally.


# References

- [Code and test Azure Functions locally](https://docs.microsoft.com/en-us/azure/azure-functions/functions-develop-local)
- [Use the Azurite emulator for local Azure Storage development](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-azurite)