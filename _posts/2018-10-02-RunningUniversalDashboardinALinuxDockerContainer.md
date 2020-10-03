---
layout: post
title: Running Universal Dashboard in a Linux Docker Container
categories: [PowerShell]
tags: [PowerShell]
---

For a project I'm currently working on I needed a Linux Docker container to run in Azure App Service for Linux.

**Web App** is a fully managed compute platform that is optimized for hosting websites and web applications. Customers can use App Service on Linux to host web apps natively on Linux for supported application stacks.

App Service on Linux supports a number of Built-in images in order to increase developer productivity. If the runtime your application requires is not supported in the built-in images, there are instructions on how to build your own Docker image to deploy to Web App for Containers.

I wanted to build my own Docker image and deploy that in later steps to the Web App for Containers.

In my opinion a good example for my custom Docker image is a simple Hello World Web page serviced by the PowerShell Module Universal Dashboard from <a href="https://twitter.com/adamdriscoll" target="_blank">Adam Driscoll</a>.

## Universal Dashboard
Universal Dashboard is a cross-platform PowerShell module for developing and hosting web-based interactive dashboards.

To create a Linux Docker container image with a Universal Dashboard I used the Ubuntu 18.04 PowerShell Core Docker image as a base for my dockerfile and added the Universal Dashboard Module and <a href="https://gist.github.com/stefanstranger/cb74f5d78d7f4111c6c66915bc89a35f" target="_blank">Universal Dashboard Hello World PowerShell script</a>.

## High-Level steps:
1. Create a <a href="https://gist.github.com/stefanstranger/cb74f5d78d7f4111c6c66915bc89a35f" target="_blank">Universal Dashboard Hello World script</a>.
2. Create a dockerfile to create the custom Docker image.
3. Create Docker image from dockerfile.
4. Start Docker container.


**Step 1. Create Universal Dashboard script.**

Please look at the simple Hello World script stored as <a href="https://gist.github.com/stefanstranger/cb74f5d78d7f4111c6c66915bc89a35f" target="_blank">Gist</a>.

**Step 2: Create a dockerfile.**
```dockerfile
FROM mcr.microsoft.com/powershell:6.1.0-rc.1-ubuntu-18.04
RUN pwsh -c "Install-Module universaldashboard -Acceptlicense -Force"
RUN pwsh -c "Invoke-WebRequest -Uri 'https://gist.githubusercontent.com/stefanstranger/cb74f5d78d7f4111c6c66915bc89a35f/raw/21a0305027495dd92d5b24e86370a3a5e5d8cdb0/HelloWorldUD-Example.ps1' -Method Get -OutFile /tmp/helloworldud-example.ps1"

CMD [ "pwsh","-command","& ./tmp/helloworldud-example.ps1" ]
```

**Step 3. Create Docker image from dockerfile.**

Open a PowerShell session and make sure you have installed the Docker for Windows Client if you are running Windows 10.

Remark:
If you are using the Docker for Windows Client make sure you have switched to Linux Containers in the settings.

```powershell
# Create Universal Dashboard Container image
docker build --rm -f "universaldashboard\dockerfile" -t universaldashboard:latest universaldashboard
```

**Step 4: Start Docker container.**

Open a PowerShell session and make sure you have installed the Docker for Windows Client if you are running Windows 10.

```powershell
#run Universal Dashboard Container.
docker run -d -p 8585:8585 --name ud universaldashboard:latest
```

Your Universal Dashboard Hello World web page is now available on port 8585 and accessible in your browser.

![Universal Dashboard running in Docker container](/assets/uddocker.gif)






**References:**
* <a href="https://docs.microsoft.com/en-us/azure/app-service/containers/app-service-linux-intro" target="_blank">Introduction to Azure App Service on Linux</a>
* <a href="https://docs.microsoft.com/en-us/azure/app-service/containers/tutorial-custom-docker-image" target="_blank">Use a custom Docker image for Web App for Containers</a>
* <a href="https://universaldashboard.io/" target="_blank">Universal Dashboard</a>
* <a href="https://hub.docker.com/r/microsoft/powershell/" target="_blank">Powershell Core Docker image</a>
* <a href="https://gist.github.com/stefanstranger/45d8129758eb44b44683c14dba2b8c45" target="_blank">Hello World Universal Dashboard script Gist</a>
