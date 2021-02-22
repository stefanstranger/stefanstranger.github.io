---
layout: post
title: IP Address Management in Enterprise Scale Landing Zones - Part 2
categories: [Azure, Azure Functions IP Addresses, Landing Zones]
tags: [Azure, Azure Policy, Governance, Development]
comments: true
---

As promised in our previous blogpost, we would write a second part in which we aim to put theory into practice. For that reason, this blogpost discusses the problems of FOO, an imaginary insurance company, regarding its network implementation and how the solutions presented in our previous blogpost will help them overcome their challenges.

To better understand the sections below, we suggest you to first read our earlier blogposts on Enterprise-Scale. Links to these blogposts can be found in the upcoming section.

- [Documentation](#documentation)
- [Foo - an Insurance Company](#foo---an-insurance-company)
- [Introduction to the MyClaims Application Team](#introduction-to-the-myclaims-application-team)
- [What problems does FOO experience?](#what-problems-does-foo-experience)
- [How to overcome the challenges of FOO?](#how-to-overcome-the-challenges-of-foo)
  - [The IP Address Management solution](#the-ip-address-management-solution)
  - [Azure IP Address Solution (IAPAS) REST API reference](#azure-ip-address-solution-iapas-rest-api-reference)
    - [Add Address Space](#add-address-space)
      - [Examples](#examples)
    - [Register Address Space](#register-address-space)
      - [Examples](#examples-1)
    - [Update Address Space](#update-address-space)
  - [Virtual Network deployment](#virtual-network-deployment)
  - [The Azure Network Policies](#the-azure-network-policies)

# Documentation

- [Blogpost: IP Address Management in Enterprise Scale Landing Zones](https://stefanstranger.github.io/2021/01/30/IPAddressManagementInESLZ/)
- [Blogpost: Becoming an Enterprise-Scale Subject Matter Expert](https://stefanstranger.github.io/2020/08/28/BecomingAnEnterpriseScaleSME/)
- [Blogpost: Enterprise-Scale – Policy Driven Governance](https://stefanstranger.github.io/2020/08/28/EnterpriseScalePolicyDrivenGovernance/)
- [Blogpost: Enterprise-Scale – Subscription Democratization](https://stefanstranger.github.io/2020/10/16/EnterpriseScaleSubscriptionDemocratization/)
- [Enterprise-Scale – Design Principles](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/enterprise-scale/design-principles)
- [Enterprise-Scale – Reference Implementation](https://github.com/Azure/Enterprise-Scale/blob/main/docs/reference/contoso/Readme.md)
- [AIPAS (Azure IP Address Solution).](https://github.com/stefanstranger/AIPAS)

# Foo - an Insurance Company

FOO operates in the insurance sector and is currently trying to become more agile by embracing DevOps practices throughout the entire company. As a result of this transformation, the executive board of FOO hopes to reduce the time-to-market of its products to stay ahead of its competitors.

As the IT department plays an important role in the transformation journey of FOO, the CIO has decided to follow the guidance of Microsoft and adopt the concept of Enterprise Scale Landing Zones (ESLZ). As discussed in one of our previous blogposts, FOO rolled out the [Wingtip](https://github.com/Azure/Enterprise-Scale/blob/main/docs/reference/wingtip/README.md) reference architecture so that they would better understand Enterprise-Scale. However, as their cloud journey has gained quit some traction, the IT department decided to implement the [Contoso](https://github.com/Azure/Enterprise-Scale/blob/main/docs/reference/contoso/Readme.md) reference architecture instead as it allows them to set up hybrid connectivity with on-premises. Figure 1 visualizes the architecture that has been deployed in Microsoft Azure.

![Visualizes the architecture that has been deployed in Microsoft Azure](/assets/19-02-2021-01.png)

*Figure 1: Contoso Reference Implementation for FOO*

At the moment, the IT department of FOO is onboarding several application teams by creating Landing Zones according to the principle of Subscription Democratization. As you might remember, this design principle supports the use of Subscriptions as a unit of management and scale. For that reason, a Subscription has also been deployed for one application team within the Claims department. Within the next section, we will elaborate on the MyClaims application this team is currently building on Azure.

# Introduction to the MyClaims Application Team

One of the first applications that FOO wants to build on Azure is called MyClaims. This application enables FOO’s customers to submit insurance claims using any device. Claims can be filed by uploading pictures and providing a description of the damage that has been sustained. In Figure 2, the solution design for this application is displayed.

![Solution design for this application is displayed](/assets/19-02-2021-02.png)

*Figure 2: Solution Design of the MyClaims Application*

As you can see, the MyClaims application is divided into a client (App Service), business (Function App) and data layer (Storage Account and SQL Database) to optimize for scalability, performance, and data integrity. Next to that, customers can interact with the front-end of the service, using any device, while common exploits and vulnerabilities are mitigated using Azure Front Door.

Furthermore, the application team wants to keep traffic between the client, business, and data layers internal. Therefore, the App Service and Function App run in an App Service Environment, while connectivity to the Storage Account and SQL Database is only possible via their respective Private Endpoints. Since similar patterns are likely to be adopted across the cloud environment of FOO, the platform (connectivity) team needs to be able to deploy private networks, compliant with all security standards, on scale. Let’s see whether they are up for the challenge!

# What problems does FOO experience?

After some weeks of investigation at FOO, we found out that the platform (connectivity) team needs to deal with several challenges before application teams can smoothly migrate to Microsoft Azure.

First, most applications within FOO have the requirement that traffic between the client, business and data layers needs to be private. In other words, most Azure Services need to be hosted in Virtual Networks. According to the platform (connectivity) team, this will ultimately result in IP space limitations across the cloud environment of FOO. For that reason, it is important that private IP space is managed very well.

Second, the platform (connectivity) team will be responsible for all these private network deployments. As every application team receives its own Landing Zone, hundreds of Subscriptions and Virtual Networks need to be rolled out in a very limited timeframe. Moreover, the Subnets within these Virtual Networks need to be protected using a pre-defined Network Security Group and Route Table. To make matters worse, the executive board has decided that the platform (connectivity) team should not expect any additional funding on the short-term as the results in the last quarters were rather disappointing. In conclusion, the workload increases substantially whereas the workforce remains constant for the foreseeable future.

Finally, the platform (connectivity) team needs to strike a clear balance between freedom and compliance regarding the network deployments of application teams. Put differently, application teams need to be able to create Subnets but these need to be protected by a standard set of Network Security Groups and Route Tables that match the network topologies of the organization. For instance, when an application team creates a Subnet to host their App Service solution, this Subnet needs to be associated with the Network Security Group and Route Table that is meant for these sorts of workloads.

# How to overcome the challenges of FOO?

The upcoming sections will clearly describe how the challenges of FOO can be overcome by the solutions presented in our previous blogpost. Next to that, we aim to provide you with all the technical details on how to build and use these solutions yourself. Therefore, all the code examples we use can be found in this public [GitHub Repository]().

## The IP Address Management solution

With the IP Address Management (IPAM) solution in place, the platform (connectivity) team can easily manage IP address ranges required for the deployment of a Landing Zone as it contains a Subscription and Virtual Network. To do so, the REST API of the IPAM solution can be used to automate a lot of the work of the platform (connectivity) team, thereby reducing its workload substantially. In Figure 3, the IPAM solution is visualized in more detail.

The REST API’s can be used to automate much of the work of the platform (connectivity) team without having a big impact on the operational activities of that team.

![IP Address Management Solution](/assets/19-02-2021-03.png)

*Figure 3: IP Address Management Solution*

Up until now, we have only shared how the IPAM solution is supposed to work and the benefits it will bring FOO. However, as we also want to enable you to use it, the next section is dedicated to interacting with the solution itself.

## Azure IP Address Solution (IAPAS) REST API reference

### Add Address Space

This REST API call can be used to add new IP address ranges to the existing Ip address ranges. As such, the platform (connectivity) team can use this REST API call to provide a representative picture of the available IP address ranges that can be used by DevOps teams. 

| AddAddressSpace |
|----------|
| HTTPS |
| POST https://aipas.azurewebsites.net/api/AddAddressSpace?code=[code] |

#### Examples

Sample Request

| AddAddressSpace |
|----------|
| HTTPS |
| POST https://aipas.azurewebsites.net/api/AddAddressSpace?code=[code] |

Request Body

<table>
  <td width="601" valign="top" style="width: 450.8pt; padding: 5px;">
  <p class="MsoNormal" style="margin-bottom:0in;line-height:normal"><span lang="EN-US">{<br></span><span style="font-size: 0.875rem;">&nbsp; &nbsp;&nbsp;</span><span style="font-size: 0.875rem;">"NetworkAddress": [<br></span><span style="font-size: 0.875rem;">&nbsp; &nbsp;&nbsp;</span><span style="font-size: 0.875rem;">"10.0.0.0/16",<br></span><span style="font-size: 0.875rem;">&nbsp; &nbsp;&nbsp;</span><span style="font-size: 0.875rem;">"10.1.0.0/16",<br></span><span style="font-size: 0.875rem;">&nbsp; &nbsp;&nbsp;</span><span style="font-size: 0.875rem;">"10.2.0.0/24"<br></span><span style="font-size: 0.875rem;">&nbsp; &nbsp;</span><span style="font-size: 0.875rem;">]<br></span><span style="font-size: 0.875rem;">}</span></p>
  </td>
</table>

Sample Response

Status code: 200

<table>
<td>
<p>[<br />&nbsp; {<br />&nbsp;&nbsp;&nbsp; "odata.metadata": "https://jpupa3c3pvbnqstorage.table.core.windows.net/$metadata#ipam/@Element",<br />&nbsp;&nbsp;&nbsp; "odata.etag": "W/\"datetime'2021-02-13T17%3A20%3A18.4783595Z'\"",<br />&nbsp;&nbsp;&nbsp; "PartitionKey": "ipam",<br />&nbsp;&nbsp;&nbsp; "RowKey": "d9c8e7e6-9840-42b8-b542-69340ba91e0f",<br />&nbsp;&nbsp;&nbsp; "Timestamp": "2021-02-13T17:20:18.4783595Z",<br />&nbsp;&nbsp;&nbsp; "CreatedDateTime": "2021-02-13T18:20:17.2578711+01:00",<br />&nbsp;&nbsp;&nbsp; "Allocated": "False",<br />&nbsp;&nbsp;&nbsp; "NetworkAddress": "10.0.0.0/16",<br />&nbsp;&nbsp;&nbsp; "FirstAddress": "10.0.0.4",<br />&nbsp;&nbsp;&nbsp; "LastAddress": "10.0.255.254",<br />&nbsp;&nbsp;&nbsp; "Hosts": 65531.0,<br />&nbsp;&nbsp;&nbsp; "LastModifiedDateTime": "2021-02-13T18:20:17.2579707+01:00"<br />&nbsp; },<br />&nbsp; {<br />&nbsp;&nbsp;&nbsp; "odata.metadata": "https://jpupa3c3pvbnqstorage.table.core.windows.net/$metadata#ipam/@Element",<br />&nbsp; &nbsp; "odata.etag": "W/\"datetime'2021-02-13T17%3A20%3A18.6027618Z'\"",<br />&nbsp; &nbsp; "PartitionKey": "ipam",<br />&nbsp;&nbsp;&nbsp; "RowKey": "5ba73975-7755-49cd-aa19-5c80395a38eb",<br />&nbsp;&nbsp;&nbsp; "Timestamp": "2021-02-13T17:20:18.6027618Z",<br />&nbsp;&nbsp;&nbsp; "CreatedDateTime": "2021-02-13T18:20:17.3911336+01:00",<br />&nbsp;&nbsp;&nbsp; "Allocated": "False",<br />&nbsp;&nbsp;&nbsp; "NetworkAddress": "10.1.0.0/16",<br />&nbsp;&nbsp;&nbsp; "FirstAddress": "10.1.0.4",<br />&nbsp;&nbsp;&nbsp; "LastAddress": "10.1.255.254",<br />&nbsp;&nbsp;&nbsp; "Hosts": 65531.0,<br />&nbsp;&nbsp;&nbsp; "LastModifiedDateTime": "2021-02-13T18:20:17.3912494+01:00"<br />&nbsp; },<br />&nbsp; {<br />&nbsp;&nbsp;&nbsp; "odata.metadata": "https://jpupa3c3pvbnqstorage.table.core.windows.net/$metadata#ipam/@Element",<br />&nbsp;&nbsp;&nbsp; "odata.etag": "W/\"datetime'2021-02-13T17%3A20%3A18.7357861Z'\"",<br />&nbsp;&nbsp;&nbsp; "PartitionKey": "ipam",<br />&nbsp;&nbsp;&nbsp; "RowKey": "7365fbe3-adf7-4a1a-a1b7-678302140b15",<br />&nbsp;&nbsp;&nbsp; "Timestamp": "2021-02-13T17:20:18.7357861Z",<br />&nbsp;&nbsp;&nbsp; "CreatedDateTime": "2021-02-13T18:20:17.5215624+01:00",<br />&nbsp;&nbsp;&nbsp; "Allocated": "False",<br />&nbsp;&nbsp;&nbsp; "NetworkAddress": "10.2.0.0/24",<br />&nbsp;&nbsp;&nbsp; "FirstAddress": "10.2.0.4",<br />&nbsp;&nbsp;&nbsp; "LastAddress": "10.2.0.254",<br />&nbsp;&nbsp;&nbsp; "Hosts": 251.0,<br />&nbsp;&nbsp;&nbsp; "LastModifiedDateTime": "2021-02-13T18:20:17.5217189+01:00"<br />&nbsp; }<br />]</p>
</td>
</table>

Status code: 400
<table>
<td>
Address Space 10.0.0.0/16 already added
</td>
</table>

### Register Address Space

This REST API call can be used by DevOps teams to claim a specific IP address range. Consequently, DevOps teams can register an IP address range themselves without the need for interaction with the platform (connectivity) team.

| RegisterAddressSpace |
|----------|
| HTTPS |
| POST https://aipas.azurewebsites.net/api/RegisterAddressSpace?code=[code] |

#### Examples

Sample Request

| RegisterAddressSpace |
|----------|
| HTTPS |
| POST https://aipas.azurewebsites.net/api/RegisterAddressSpace?code=[code] |

Request Body

<table>
  <td>
  <p>{<br />&nbsp;&nbsp; "InputObject": {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "ResourceGroup": "myclaims-rg",<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "VirtualNetworkName": "MyClaims-vnet"<br />&nbsp; }<br />}</p>
  </td>
</table>

Sample Response

Status code: 200

<table>
<td><p>{&nbsp;<br />&nbsp; "odata.type": "jpupa3c3pvbnqstorage.ipam",&nbsp;<br />&nbsp; "odata.id": "https:// jpupa3c3pvbnqstorage.table.core.windows.net/ipam(PartitionKey='ipam',RowKey='4fb91aa3-ec73-49cc-a644-9d02ab7a82cf')",&nbsp;<br />&nbsp; "odata.etag": "W/\"datetime'2021-01-24T14%3A53%3A14.4436468Z'\"",&nbsp;<br />&nbsp; "odata.editLink": "ipam(PartitionKey='ipam',RowKey='4fb91aa3-ec73-49cc-a644-9d02ab7a82cf')",&nbsp;<br />&nbsp; "PartitionKey": "ipam",&nbsp;<br />&nbsp; "RowKey": "4fb91aa3-ec73-49cc-a644-9d02ab7a82cf",&nbsp;<br />&nbsp; "Timestamp@odata.type": "Edm.DateTime",&nbsp;<br />&nbsp; "Timestamp": "2021-01-24T14:53:14.4436468Z",&nbsp;<br />&nbsp; "Allocated": "False",&nbsp;<br />&nbsp; "CreatedDateTime": "2021-01-15T13:56:07.5253887+01:00",&nbsp;<br />&nbsp; "FirstAddress": "10.2.0.4",&nbsp;<br />&nbsp; "Hosts": 251.0,&nbsp;<br />&nbsp; "LastAddress": "10.2.0.254",&nbsp;<br />&nbsp; "LastModifiedDateTime": "2021-01-24T15:44:14.8422325+01:00",&nbsp;<br />&nbsp; "NetworkAddress": "10.2.0.0/24"&nbsp;<br />}&nbsp;</p>
</td>
</table>

Status code: 400
<table>
<td>
Failed to register free address space
</td>
</table>

### Update Address Space

The UpdateAddressSpaceTimer Azure Function is a timer function which is scheduled to run every day at 9:30 AM. If you want to change the schedule please change the [function.json](https://github.com/stefanstranger/AIPAS/blob/master/src/function/UpdateAddressSpaceTimer/function.json) schedule configuration.

## Virtual Network deployment

After a DevOps team makes a REST API call to the RegisterAddressSpace Azure Function, an IP address range is reserved for that team. Subsequently, this IP address range can be used when creating a new Virtual Network as part of the Landing Zone deployment. 

So, suppose that you want to create a Landing Zone according to the configuration below.

| Azure Resource | Value |
|----------|----------|
| Subscription name | Landing Zone |
| ResourceGroup name | myclaims-rg |
| ResourceGroup location | westeurope | 
| Virtual Network Name | MyClaims-vnet |

The following PowerShell script can be used to register an IP address space using the RegisterAddressSpace Azure Function, create a Resource Group and finally use the registered IP address space to create a Virtual Network in that Resource Group.

```PowerShell
# Create Resource Group and Vnet example PowerShell script

#region variables
$ResourceGroupName = 'myclaims-rg'
$VNetName = 'MyClaims-vnet'
$Location = 'westeurope'
#endregion

#region Call Registere Address Space REST API

$uri = 'https://ipam.azurewebsites.net/api/RegisterAddressSpace?code=[yourcode]'
$body = @{
    'InputObject' = @{
        'ResourceGroup' = $ResourceGroupName
        'VirtualNetworkName' = $VNetName
    }
} | ConvertTo-Json

$params = @{
    'Uri'         = $uri
    'Method'      = 'POST'
    'ContentType' = 'application/json'
    'Body'        = $Body
}

$Result = Invoke-RestMethod @params
#endregion

#region Create RG and Vnet
New-AzResourceGroup -Name $ResourceGroupName -Location $Location

New-AzVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroupName -Location $Location -AddressPrefix $Result.NetworkAddress
#endregion
```

In this way, the provisioning of a Landing Zone becomes more efficient for both the DevOps teams and platform (connectivity) team. Moreover, when implementing the Azure Network Policies as described in the next section, this process also becomes more controllable for the platform (connectivity) team. 

## The Azure Network Policies

As mentioned before, the platform (connectivity) team of FOO must deal with multiple challenges. In our opinion, Azure Policy can help to overcome the second and third challenge of FOO as this service can be used to enforce multiple networking topologies while reducing the burden on your team. The reason behind this is that Azure Policy can append the correct Network Security Group and Route Table to a Subnet upon creation, based on its networking topology. Unfortunately, there is no option to define a type for a specific Subnet, meaning that you should resort to a naming convention instead. For that reason, we have created the [Subnet-Naming-Convention](https://github.com/stefanstranger/AIPAS/blob/master/policies/Subnet-Naming-Convention.json) Policy Definition. When assigned, this Policy Definition ensures that Subnets can only contain the string ‘app’ as the deployment will fail otherwise as you can see in Figure 4.

![Deployment failed due to the Subnet-Naming-Convention Policy Definition](/assets/19-02-2021-04.png)

*Figure 4: Deployment failed due to the Subnet-Naming-Convention Policy Definition*

On the other hand, the creation of a Subnet using the correct naming convention (e.g. MyClaims-app-snet), will result in a successful deployment as is visualized in Figure 5.

![Deployment succeeded when using the correct Naming Convention](/assets/19-02-2021-05.png)

*Figure 5: Deployment succeeded when using the correct Naming Convention*

Next to that we deployed the [Subnet-Route-Table](https://github.com/stefanstranger/AIPAS/blob/master/policies/Subnet-Route-Table.json) and [Subnet-Network-Security-Group](https://github.com/stefanstranger/AIPAS/blob/master/policies/Subnet-Network-Security-Group.json) Policy Definitions. After creating a Policy Assignment for both Policy Definitions, each Subnet for which the name property contains ‘app’, will have the correct Network Security Group (e.g. app-network-security-group) and Route Table (e.g. app-route-table) associated to it. Figure 6 displays the state of the Subnet after it has been deployed and the Policy Assignments have appended the correct objects to it.

![The Subnet with the correct Network Security Group and Route Table associated to it](/assets/19-02-2021-06.png)

*Figure 6: The Subnet with the correct Network Security Group and Route Table associated to it*

In this way, the platform (connectivity) team of FOO can enforce the correct networking topology while not having to do all the heavy lifting themselves. In other words, the use of Azure Policy has largely solved their second and third challenge, making them more confident to start migrating the rest of the workloads to Microsoft Azure.

Nevertheless, the platform (connectivity) team needs to improve the Policy Definitions to accommodate for a broader range of scenarios. For instance, the Subnet-Naming-Convention should also support other string values, such as ‘aks’ (for Azure Kubernetes Services) and ‘vm’ (for Virtual Machines), if these networking topologies are required by the application teams. Consequently, the Subnet-Network-Security-Group and Subnet-Route-Table Policy Definitions need to be improved as well to support these scenarios and assign the right Network Security Group and Route Table to the correct Subnet. In other words, there is still a lot of work to be done!

We hope you enjoyed our new blogpost,

[Bas van Bennekom](https://www.linkedin.com/in/bas-van-bennekom) and [Stefan Stranger](https://www.linkedin.com/in/stefanstranger/)