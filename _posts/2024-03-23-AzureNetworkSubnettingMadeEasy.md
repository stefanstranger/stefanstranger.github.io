---
layout: post
title: Azure Network Subnetting made easy
categories: [Networking, Azure]
tags: [Networking, Azure]
comments: true
---

- [Introduction](#introduction)
- [What is Azure Subnet Copilot?](#what-is-azure-subnet-copilot)
- [How Does Azure Subnet Copilot Work?](#how-does-azure-subnet-copilot-work)
- [Online version](#online-version)
- [Usage scenario's](#usage-scenarios)
  - [Network knowledge](#network-knowledge)
  - [Programmatic use of the solution](#programmatic-use-of-the-solution)
- [Conclusion](#conclusion)
- [References](#references)

## Introduction

Managing IP addresses in Azure Virtual Network can be a challenging task, especially for those who don't have a deep understanding of networking. That's why I've developed **Azure Subnet Copilot**, a solution designed to simplify this process. In this blog post, I'll introduce Azure Subnet Copilot, explain how it works, and show you how you can use it to manage your Azure Virtual Network Subnets in a simplified and more effective way.

## What is Azure Subnet Copilot?

Azure Subnet Copilot is a user-friendly IP management solution. Unlike existing subnet calculators, which require some basic network knowledge, Azure Subnet Copilot is designed to be user-friendly. It asks for the required `number of IP addresses` for which you want the subnet to be created, and it does the rest for you. Based on the Azure Virtual Network IP Range and any already existing subnet IP ranges it will calculate the next smallest available ip range (in CIDR format) suitable for the number of IP addresses you need.

One of the key features of Azure Subnet Copilot is that it takes into account the Azure reserved IP addresses. This ensures that you have sufficient IP addresses available for your solution that you want to deploy in Azure. 

## How Does Azure Subnet Copilot Work?

Azure Subnet Copilot is developed in Python to calculate the appropriate subnet based on the number of IP addresses you need. It uses a number of Python libraries to perform the subnet calculations, and it uses Flask to provide a user-friendly web interface.

When you enter the number of IP addresses you need into the web interface, Azure Subnet Copilot calculates the smallest possible subnet that can accommodate that number of IP addresses. It then displays the subnet in CIDR notation, along with the range of usable IP addresses in that subnet.

## Online version

There is an online version available at: <a href="https://azure-subnet-copilot.vercel.app/" target="_blank">https://azure-subnet-copilot.vercel.app/</a>

![Animated gif of website with Azure Subnet Copilot input and output being shown](/assets/2024-03-23-01.gif)

The solutions requires the following information to be provided:

- Virtual Network IP range (in CIDR format)
- Existing Subnet IP ranges (in CIDR format) separated by comma's (or leave empty if no subnet are deployed)
- Number of required IP addresses

## Usage scenario's

One of the scenario's in mind while developing this solution was a DevOps team who has received an Azure Landing Zone Subscription and wants to deploy their solution in Azure using Continuous Integration and Continuous Deployment pipelines.

Often these Azure Landing Zone Subscriptions are deployed by a central (Azure) Platform team who also deploys an Azure Virtual Network within the assigned Azure Subscription. The Virtual Network is often managed by the platform team, but the DevOps team is allowed to deploy their own Azure Virtual Subnets within the already deployed Azure Virtual Network using Infrastructure as Code via CI/CD pipelines. This what is called <a href="https://stefanstranger.github.io/2020/10/16/EnterpriseScaleSubscriptionDemocratization" target="_blank">Subscription Democratization</a> according to the [Cloud Adaption Framework](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/).

With this solution it's very easy for Application DevOps teams to deploy an Azure Virtual Subnet to support their application deployment in Azure, by just providing the required IP addresses.

### Network knowledge

For the deployment of an Azure Virtual Subnet to support the application hosted in an Azure Landing Zone Subscription, you need to have some basic network subnetting knowledge.

Subnets enable you to segment the virtual network into one or more subnetworks and allocate a portion of the virtual network's address space to each subnet. You can then deploy Azure resources in a specific subnet. Just like in a traditional network, subnets allow you to segment your virtual network address space into segments that are appropriate for the organization's internal network. Segmentation improves address allocation efficiency. 

When creating a new Azure Virtual Network subnet you need to provide the subnet mask. A Subnet Mask is a 32-bit number used to distinguish the network and host portions of an IP address.

For defining the Subnet address range a (Classless Inter-Domain Routing) CIDR notation is used. It subdivides IP addresses into smaller blocks (CIDR blocks) based on the number of available bits.

CIDR notation represents the number of bits in the prefix. For example:

- 10.0.0.0/29 means the first 29 bits are the network prefix, leaving 3 bits for hosts (equivalent to subnet mask 255.255.255.248).

In a /29 subnet, there are 32 - 29 = 3 host bits available. The formula to calculate the number of hosts is 2 raised to the power of the number of host bits minus 2. The subtraction of 2 is because in IPv4 addressing, the first and last addresses in a subnet are reserved: the first for the network address and the last for the broadcast address.

So, for a /29 subnet:

Number of hosts = (2^3) - 2
= 8 - 2
= 6

Therefore, the CIDR notation 10.0.0.0/29 allows for 6 hosts. In the case of **Azure** you need to add 3 extra addresses that cannot be used. So 8 - 5 = 3 available addresses.

So when you need a subnet for 3 ip addresses you need to understand that you need a the following subnet (/29) to create in the Azure Portal.

![Screenshot of Azure Portal to create Azure Virtual Network Subnet](/assets/2024-03-23-02.png)

And you also need to take into account any already existing subnets when calculating the correct subnet address range.

And this is exactly why I created the Azure Subnet IP range finder (aka Azure Subnet Copilot).

With the Azure Subnet IP range finder, you just need to provide the Azure Virtual Network Address Space.

![Screenshot of Azure Virtual Network Address space in the Azure Portal](/assets/2024-03-23-03.png)

And (if any) existing Azure Network Subnets.  

![Screenshot of Azure Virtual Network Subnets in the Azure Portal](/assets/2024-03-23-04.png)

So suppose you need another Azure Virtual Subnet with 16 ip addresses in the existing Azure Virtual Network with IP range 10.0.0.0/24 and the existing subnet with ip range 10.0.0.0/29, you can insert this information in the online solution and get a suitable ip range for the subnet to be used.

![Screenshot of Online Solution](/assets/2024-03-23-05.png)

Output of online solution.

![Screenshot of Online Solution result.](/assets/2024-03-23-06.png)

We can then use this suitable ip range in the Azure Portal for the new Subnet.

![Screenshot of Azure Portal with new Azure Virtual Network Subnet information](/assets/2024-03-23-07.png)

**Notes:**

Because we needed 16 ip addresses we need a minimum of 16 + 5 addresses that's why the first available range is 10.0.0.32/27. If we only needed 8 (8  + 5 = 13) we would have got 10.0.0.16/28 back as a suitable ip range.

### Programmatic use of the solution

The solution can be used via the interactive online version or you can use this in a programmatic way by calling the solution via a REST API call from for instance PowerShell.

By using the solution from the command prompt it allows Application DevOps teams to integrate this into their CICD pipelines and only having to worry about the required IP addresses.

Below PowerShell script is an example on how you can use the solution from the PowerShell command prompt.

<iframe frameborder="0" scrolling="no" style="width:100%; height:2215px;" allow="clipboard-write" src="https://emgithub.com/iframe.html?target=https%3A%2F%2Fgithub.com%2Fstefanstranger%2FAzureSubnetCopilot%2Fblob%2Fmain%2Fexamples%2FPowerShell-Example.ps1&style=default&type=code&showBorder=on&showLineNumbers=on&showFileMeta=on&showFullPath=on&showCopy=on"></iframe>

## Conclusion

Azure Subnet Copilot is a powerful tool that can simplify the process of managing Subnets in Azure Virtual Network. Whether you're a networking expert or a beginner, I believe that Azure Subnet Copilot can make your life easier. I encourage you to give it a try and see how it can help you manage your IP addresses more effectively.

If you want to host Azure Subnet Copilot in your own environment, you can! All the code used to deploy this solution is available on <a href="https://github.com/stefanstranger/AzureSubnetCopilot" target="_blank">GitHub</a>. If you encounter any issues or have any suggestions, please submit them on <a href="https://github.com/stefanstranger/AzureSubnetCopilot" target="_blank">my GitHub page</a>. Your feedback is greatly appreciated!

## References

- <a href="https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-faq#basics" target="_blank">Azure Virtual Network Basics</a>
- [Understand TCP/IP addressing and subnetting basics](https://learn.microsoft.com/en-us/troubleshoot/windows-client/networking/tcpip-addressing-and-subnetting)
- [Github Repository - AzureSubnetCopilot](https://github.com/stefanstranger/AzureSubnetCopilot)