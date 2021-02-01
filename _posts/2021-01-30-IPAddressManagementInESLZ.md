---
layout: post
title: IP Address Management in Enterprise Scale Landing Zones
categories: [Azure, Azure Functions IP Addresses, Landing Zones]
tags: [Azure, Azure Policy, Governance, Development]
comments: true
---

Within this blog post, we discuss the implementation of an IP Address Management Solution (IPAM) and Policy Driven Governance. As such, we want to show how Enterprise-Scale can support Subscription Democratization by providing DevOps teams with the possibility to create their own Subnets that suit the needs of their application while enforcing compliance and security on the platform level.
To fully understand this blog post, and the terminology used within, we recommend you to first read our earlier blog posts on Enterprise-Scale which can be found in the upcoming section.

- [Documentation](#documentation)
- [Azure Virtual Networks](#azure-virtual-networks)
- [Enterprise-Scale IPAM Solution](#enterprise-scale-ipam-solution)
- [Custom IPAM Solution](#custom-ipam-solution)
  - [Azure Function](#azure-function)
    - [AddAddressSpace](#addaddressspace)
    - [RegisterAddressSpace](#registeraddressspace)
    - [UpdateAddressSpace](#updateaddressspace)
  - [Storage Table](#storage-table)
- [Enforcing a Network Topology using Azure Policy](#enforcing-a-network-topology-using-azure-policy)
- [Putting Theory into Practice](#putting-theory-into-practice)

## Documentation

* [Blogpost: Becoming an Enterprise-Scale Subject Matter Expert](https://stefanstranger.github.io/2020/08/28/BecomingAnEnterpriseScaleSME/)
* [Blogpost: Enterprise-Scale – Policy Driven Governance](https://stefanstranger.github.io/2020/08/28/EnterpriseScalePolicyDrivenGovernance/)
* [Blogpost: Enterprise-Scale – Subscription Democratization](https://stefanstranger.github.io/2020/10/16/EnterpriseScaleSubscriptionDemocratization/)
* [Enterprise-Scale – Creating Landing Zones](https://github.com/Azure/Enterprise-Scale/blob/main/docs/Deploy/enable-subscription-creation.md#create-landing-zones-subscriptions)
* <a href="https://github.com/Azure/Enterprise-Scale/blob/main/azopsreference/3fc1081d-6105-4e19-b60c-1ec1252cf560%20(3fc1081d-6105-4e19-b60c-1ec1252cf560)/ESLZ%20(ESLZ)/.AzState/Microsoft.Authorization_policyDefinitions-Deploy-vNet.parameters.json" target="_blank">Enterprise-Scale – Creating Virtual Networks</a>
* [Cloud Adoption Framework – Network Topology and Connectivity](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/enterprise-scale/network-topology-and-connectivity?branch=pr-en-us-1185)
* [Azure Virtual Networks – Concepts and Best Practices](https://docs.microsoft.com/en-us/azure/virtual-network/concepts-and-best-practices)
* [Deploy Enterprise-Scale with hub and spoke architecture](https://github.com/Azure/Enterprise-Scale/tree/main/docs/reference/adventureworks)


## Azure Virtual Networks

Azure Virtual Networks are a fundamental building block for private networks in Azure. Virtual Networks enable different types of Azure Services, such as Virtual Machines, to securely communicate with each other, the internet, and on-premises networks. When creating a Virtual Network, you must specify a custom private IP address space using public and private (RFC 1918) addresses. Azure assigns resources in a Virtual Network a private IP address from the address space that you assign it. For example, if you deploy a Virtual Machine in a Virtual Network with IP address space 10.0.0.0/16, the VM will be assigned a private IP address of 10.0.0.4. 

Subnets are a component of Virtual Networks and enable you to segment it into one or more sub-networks. Consequently, you can allocate a subnet of the Virtual Network's IP address space to each Subnet. Subsequently, you can deploy Azure Services your newly created Subnet. 

If we consider the Cloud Adaption Framework (CAF) documentation concerning network topology and connectivity, planning for IP addressing in Microsoft Azure is deemed very important as you need to ensure that IP address space across on-premises locations and Azure regions does not overlap.

In order to help you with this, an IPAM solution can be implemented. This solution enables you to plan, deploy and manage your IP address space. In other words, the IPAM solution enables you to keep the inventory of assignable IP addresses up-to-date and adequate for further expanding your cloud environment.

## Enterprise-Scale IPAM Solution

Enterprise-Scale supports Subscription Democratization as it provides DevOps teams with the correct Role Based Access Control (RBAC) permissions to deploy Subnets that fulfill the requirements of their application while at the same time maintaining compliance and security as defined by the Platform (SecOps) team. 

As you might remember from one of our previous blog posts, the Platform team is responsible for creating Landing Zones for DevOps teams. Within a Landing Zone, a Virtual Network is created as well and assigned IP address space to support the deployment of applications. Within Enterprise-Scale, this process is handled via an Azure Policy called <a href="https://github.com/Azure/Enterprise-Scale/blob/2fdca659c67acb865b3e6d3a7d09af2bd3011d03/azopsreference/3fc1081d-6105-4e19-b60c-1ec1252cf560 (3fc1081d-6105-4e19-b60c-1ec1252cf560)/ESLZ (ESLZ)/.AzState/Microsoft.Authorization_policyDefinitions-Deploy-vNet.parameters.json" target="_blank">Deploy-vNet</a>. This Azure Policy has a ‘DeployIfNotExists’ effect and thus ensures that every new Subscription will contain a Virtual Network with a pre-defined IP address space assigned to it.  

The <a href="https://github.com/Azure/Enterprise-Scale/blob/main/examples/landing-zones/connected-subscription/connectedSubscription.json" target="_blank">Connected Subscription ARM Template deployment</a> takes a dependency on the '<a href="https://github.com/Azure/Enterprise-Scale/blob/main/azopsreference/3fc1081d-6105-4e19-b60c-1ec1252cf560%20(3fc1081d-6105-4e19-b60c-1ec1252cf560)/ESLZ%20(ESLZ)/.AzState/Microsoft.Authorization_policyDefinitions-Deploy-VNET-HubSpoke.parameters.json" target="_blank">Deploy-VNET-HubSpoke</a>' policy provided by Enterprise-Scale reference implementations, and will invoke the template deployment in the policyDefinition as part of assigning the policy to the newly created landing zone (subscription). This policy deploys virtual network and peer to the hub. 

Finally the <a href="https://github.com/Azure/Enterprise-Scale/blob/2fdca659c67acb865b3e6d3a7d09af2bd3011d03/azopsreference/3fc1081d-6105-4e19-b60c-1ec1252cf560%20(3fc1081d-6105-4e19-b60c-1ec1252cf560)/ESLZ%20(ESLZ)/.AzState/Microsoft.Authorization_policyDefinitions-Deploy-vNet.parameters.json" target="_blank">'Deploy-VNet'</a> policy deploys a spoke network with configuration to hub network based on ipam configuration object.

As discussed above, Enterprise-Scale offers IPAM functionality when configuring and deploying Virtual Networks via an ARM template and Azure Policies. We identified the following challenges when using this approach:

* Every time a new Landing Zone is deployed, the Policy Definition (Deploy-vNet) responsible for creating a Virtual Network, needs to be updated.
* Every time a Landing Zone is removed, the IP address space of its associated Virtual Network is not dynamically released for the creation of new Landing Zones.
* he solution does not support workload-specific configurations on the Subnet in terms of Network Security Groups and Route Tables.

To address these challenges, we have created a custom IPAM solution, which will be discussed in the upcoming section.

## Custom IPAM Solution

The custom IPAM solution we developed offers the following functionality:
* IP address space can be managed by a central platform (connectivity) team.
* IP address space can be claimed by a DevOps team in a programmatic manner.
As such, the IPAM solution creates an up-to-date overview of the IP address space that is available in your organization.

![IPAM Solution Overview](/assets/IPAM%20Solution%20Overview.png)

*Figure 1 : Overview of custom IPAM Solution*

### Azure Function

The Azure Function is written in PowerShell and allows the central network platform team to create address spaces to be used during Landing Zone deployments with their vNets and store the address space records in an Azure Storage Table.

The Azure Functions work as follows.

#### AddAddressSpace

The AddAddressSpace Function enables the Platform team to add new IPAM records to the Azure Storage Table which can be used as Address Spaces for new vNet deployments. Multiple CIDR Address Spaces can be added in a single POST REST API call to the AddAddressSpace Azure Function.

A new IPAM Record will contain the following properties:
- PartitionKey
- RowKey
- TimeStamp
- Allocated
- CreatedDateTime
- FirstAddress
- Hosts
- LastAddress
- LastModifiedDateTime

#### RegisterAddressSpace

The RegisterAddressSpace Function can be used to register a free Address Space to be used when deploying a new vNet. The Function checks for the oldest (using the CreatedDateTime) free (not allocated) Address Space via a POST REST API call containing the ResourceGroup and VirtualNetworkName of the vNet for which the Address Space is to be used.

The response contains all the information required for the next step the vNet deployment.

The IPAM record is updated with Allocated property True.

#### UpdateAddressSpace

The last Function is used to update the IPAM Storage Table with vNet deployment information. When an registered Address Space is used for a vNet deployment the Allocated record is updated with the following properties:

- ResourceGroup
- Subscription
- VirtualNetworkName

This Function is a timer triggered Function.

### Storage Table

The Azure Storage Table will contain the following Entities and properties:

![Storage Table](/assets/StorageTable.png)

*Figure 2: Screenshot over Azure Storage Table with IPAM configuration*

Let's get some more context on the Azure Table properties:

|Property  | Example value | Description |
|----------|---------------|-------------|
| PartitionKey | ipam | Due to the limited number of entities one PartitionKey should be sufficient |
| RowKey | 30728a55-9af9-407f-b614-afb8a8ea4bdd | The RowKey is the primary key and for each of the entities a GUID is used as RowKey |
| Timestamp | 2021-01-31T13:29:30.712Z | The Timestamp property is maintained by server and it’s the time the entity was last modified  |
| CreatedDateTime | 2021-01-31T14:29:31.4352067+01:00 | The CreatedDateTime is a showing the time when the IPAM Address Space entity was created. This used to determine the oldest free Address Space entity |
| Allocated | True | The Allocated property shows if an IPAM Address Space entity is free (Allocated = False) or registered to be used for a vNet as address space |
| NetworkAddress | 10.0.0.0/16 | CIDR Address Space to be used for vNet Address Spaces |
| FirstAddress | 10.0.0.4 | Shows the first Address which can be used for hosts of the Address Space. Azure reserves 5 IP Addresses witin each Subnet. More info [here](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-faq#are-there-any-restrictions-on-using-ip-addresses-within-these-subnets). |
| LastAddress | 10.0.255.254 | Shows the last address to be used for the (NetworkAddress) Address Space. |
| Hosts | 65531 | Shows the number of available hosts for the (NetworkAddress) Address Space. This could be used as parameter input to select best available Address Space without loosing too much IP Addresses. Currently not used with Azure Functions. |
| LastModifiedDateTime | 2021-01-31T14:36:33.5418326+01:00 | The LastModifiedDateTime entity property shows the last time the entity was modified. This could be triggered by the UpdateAddressSpace Azure Function for instance. |
| ResourceGroup | networkwatcherrg | The ResourceGroup entity property shows the name of the Resource Group where the vNet containing the Address Space (NetworkAddress) is being used. This property is set when the UpdateAddressSpace Azure Function is triggered. |
| Subscription | Contoso | The Subscription entity property shows the name of the Azure Subscription where the vNet containing the Address Space (NetworkAddress) is being used. This property is set when the UpdateAddressSpace Azure Function is triggered |
| VirtualNetworkName | demo-vnet | The VirtualNetworkName entity property shows the name of the vNet where the Address Space (NetworkAddress) is being used. This property is set when the UpdateAddressSpace Azure Function is triggered |


## Enforcing a Network Topology using Azure Policy

In the previous section, we discussed the creation of an IPAM solution and how it will help you to provision a Landing Zone, containing a Subscription and Virtual Network. From this point onwards, ESLZ advocates that DevOps teams should be able develop and manage their solutions in an autonomous fashion.

Nevertheless, some enterprise customers might want to enforce a standard network topology at the same time. As such, the platform (connectivity) team would most likely bear the responsibility of enforcing this network topology, which will increase as your cloud environment grows. To reduce the amount of work for your platform team, while providing more freedom to your DevOps teams.

![Using Azure Policy to enforce a desired Networking Topology](/assets/AzurePolicyToEnforceNetworkPolicy.png)

*Figure 3: Using Azure Policy to enforce a desired Networking Topology*

Azure Policy can be adopted in terms of networking. In Figure 2, we have visualized how different Azure Policies (e.g. Subnet-Route-Table) can accommodate your desired network topology.

As you can see, the cloud environment consists of a Subscription and Virtual Network that were provisioned as part of a Landing Zone. Next to that, it is recommended to deploy a standard set of Network Security Groups and Route Tables for the different network topologies your company supports. Let’s say that your company uses Azure Kubernetes Services (AKS) for specific workloads and only wants outbound traffic to the internet to flow via an Azure Firewall. If this is a common network topology, you can create a Route Table, including this User Defined Route, and associate it with all Subnets in which AKS workloads will land. The same applies to Network Security Groups when you for instance only allow inbound traffic on certain ports. Therefore, you should create a standard set of Network Security Groups and Route Tables for the different type of Azure Services (e.g. Virtual Machines or App Services) you are planning to use.

However, after creating these resources, you should also ensure that they are associated with the correct Subnets and that DevOps teams cannot remove these associations. For this purpose, Azure Policy can be used. First, you need to create a Policy Definition that enforces a naming convention on Subnets, aligned with the network topologies you support, as it is necessary for associating a Network Security Group and Route Table later on. 

Policy Definition Subnet-Naming-Convention:

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "input": {
            "value": {
                "Name": "Subnet-Naming-Convention",
                "ResourceId": "/providers/Microsoft.Management/managementGroups/fscp/providers/Microsoft.Authorization/policyDefinitions/Subnet-Naming-Convention",
                "ResourceName": "Subnet-Naming-Convention",
                "ResourceType": "Microsoft.Authorization/policyDefinitions",
                "SubscriptionId": null,
                "PolicyDefinitionId": "/providers/Microsoft.Management/managementGroups/fscp/providers/Microsoft.Authorization/policyDefinitions/Subnet-Naming-Convention",
                "Properties": {
                    "Description": "This Azure Policy enforces a naming convention on Subnets for your network topology.",
                    "DisplayName": "Subnet-Naming-Convention",
                    "Mode": "All",
                    "Parameters": {
                        "effect": {
                            "type": "String",
                            "metadata": {
                                "displayName": "Allowed Azure Policy Effect",
                                "description": "The allowed effect for this Azure Policy"
                            },
                            "defaultValue": "Deny"
                        }
                    },
                    "PolicyRule": {
                        "if": {
                            "anyOf": [
                                {
                                    "allOf": [
                                        {
                                            "field": "type",
                                            "equals": "Microsoft.Network/virtualNetworks"
                                        },
                                        {
                                            "field": "Microsoft.Network/virtualNetworks/subnets[*].name",
                                            "notContains": "aks"
                                        }
                                    ]
                                },
                                {
                                    "allOf": [
                                        {
                                            "field": "type",
                                            "equals": "Microsoft.Network/virtualNetworks/subnets"
                                        },
                                        {
                                            "not": {
                                                "field": "name",
                                                "contains": "aks"
                                            }
                                        }
                                    ]
                                }
                            ]
                        },
                        "then": {
                            "effect": "[parameters('effect')]"
                        }
                    }
                }
            }
        }
    }
}
```

As you can see, the Policy Definition ensures that a Subnet name should contain the string ‘aks’. If this is not the case, the deployment will automatically fail as the effect is set to ‘Deny’. Be aware that you can expand the rule of the Policy Definition by including other string values (e.g. ‘iaas’) to also accommodate for other network topologies.

Subsequently, you can create another Policy Definition to associate the Subnet with a distinct Route Table during deployment. As a result, you can ensure that once a DevOps team deploys a Subnet, the correct Route Table is associated since the naming convention of the Subnet is used as a condition in the policy rule.

Policy Definition Subnet-Route-Table:

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "input": {
            "value": {
                "Name": "Subnet-Route-Table",
                "ResourceId": "/providers/Microsoft.Management/managementGroups/fscp/providers/Microsoft.Authorization/policyDefinitions/Subnet-Route-Table",
                "ResourceName": "Subnet-Route-Table",
                "ResourceType": "Microsoft.Authorization/policyDefinitions",
                "SubscriptionId": null,
                "PolicyDefinitionId": "/providers/Microsoft.Management/managementGroups/fscp/providers/Microsoft.Authorization/policyDefinitions/Subnet-Route-Table",
                "Properties": {
                    "Description": "This Azure Policy appends a Route Table in accordance with the naming convention of the Subnet.",
                    "DisplayName": "Subnet-Route-Table",
                    "Mode": "All",
                    "Parameters": {
                        "effect": {
                            "type": "String",
                            "metadata": {
                                "displayName": "Allowed Azure Policy Effect",
                                "description": "The allowed effect for this Azure Policy"
                            },
                            "defaultValue": "Append"
                        }
                    },
                    "PolicyRule": {
                        "if": {
                            "allOf": [
                                {
                                    "field": "type",
                                    "equals": "Microsoft.Network/virtualNetworks/subnets"
                                },
                                {
                                    "field": "name",
                                    "contains": "aks"
                                },
                                {
                                    "field": "Microsoft.Network/virtualNetworks/subnets/routeTable.id",
                                    "notEquals": "[concat(subscription().id, '/resourceGroups/', resourcegroup().name, '/providers/Microsoft.Network/routeTables/', 'aks-route-table')]"
                                }
                            ]
                        },
                        "then": {
                            "effect": "[parameters('effect')]",
                            "details": [
                                {
                                    "field": "Microsoft.Network/virtualNetworks/subnets/routeTable",
                                    "value": {
                                        "id": "[concat(subscription().id, '/resourceGroups/', resourcegroup().name, '/providers/Microsoft.Network/routeTables/', 'aks-route-table')]"
                                    }
                                }
                            ]
                        }
                    }
                }
            }
        }
    }
}

```

The policy rule of the Policy Definition evaluates if a Subnet name contains the string ‘aks’ and if it is not associated with the ‘aks-route-table’ resource. If the policy rule evaluates to true, the ‘Append’ effect ensures that the Route Table is appended to the Subnet during deployment. Another benefit of using this Azure Policy effect is that it re-appends the Route Table to the Subnet when a DevOps team tries to remove it since Azure Policy considers this action as an incremental deployment.

Finally, you should create a Policy Definition that associates the Subnet with a specific Network Security Group. This ensures that Subnets are created using a pre-defined Network Security Group which is based on the naming convention of the Subnet.

Policy Definition Subnet-Network-Security-Group:

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "input": {
            "value": {
                "Name": "Subnet-Network-Security-Group",
                "ResourceId": "/providers/Microsoft.Management/managementGroups/fscp/providers/Microsoft.Authorization/policyDefinitions/Subnet-Network-Security-Group",
                "ResourceName": "Subnet-Network-Security-Group",
                "ResourceType": "Microsoft.Authorization/policyDefinitions",
                "SubscriptionId": null,
                "PolicyDefinitionId": "/providers/Microsoft.Management/managementGroups/fscp/providers/Microsoft.Authorization/policyDefinitions/Subnet-Network-Security-Group",
                "Properties": {
                    "Description": "This Azure Policy appends a Network Security Group in accordance with the naming convention of the Subnet.",
                    "DisplayName": "Subnet-Network-Security-Group",
                    "Mode": "All",
                    "Parameters": {
                        "effect": {
                            "type": "String",
                            "metadata": {
                                "displayName": "Allowed Azure Policy Effect",
                                "description": "The allowed effect for this Azure Policy"
                            },
                            "defaultValue": "Append"
                        }
                    },
                    "PolicyRule": {
                        "if": {
                            "allOf": [
                                {
                                    "field": "type",
                                    "equals": "Microsoft.Network/virtualNetworks/subnets"
                                },
                                {
                                    "field": "name",
                                    "contains": "aks"
                                },
                                {
                                    "field": "Microsoft.Network/virtualNetworks/subnets/networkSecurityGroup.id",
                                    "notEquals": "[concat(subscription().id, '/resourceGroups/', resourcegroup().name, '/providers/Microsoft.Network/networkSecurityGroups/', 'aks-network-security-group')]"
                                }
                            ]
                        },
                        "then": {
                            "effect": "[parameters('effect')]",
                            "details": [
                                {
                                    "field": "Microsoft.Network/virtualNetworks/subnets/networkSecurityGroup",
                                    "value": {
                                        "id": "[concat(subscription().id, '/resourceGroups/', resourcegroup().name, '/providers/Microsoft.Network/networkSecurityGroups/', 'aks-network-security-group')]"
                                    }
                                }
                            ]
                        }
                    }
                }
            }
        }
    }
}

```

As is denoted in the Policy Definition, the policy rule evaluates whether the Subnet name contains the string ‘aks’ and if it is not associated with the aks-network-security-group resource. If this condition is met, the ‘Append’ effect enforces that the above-mentioned Network Security Group is appended to the Subnet upon creation. Again, the ‘Append’ effect also ensures that the Network Security Group is associated with the Subnet again even though the DevOps team might have just removed it. 

These Policy Definitions focus on the AKS resource but can also be expanded to accommodate for other Azure Services such as Virtual Machines or App Services. In this way, you can provide your DevOps teams with the freedom to create their own Subnets while you enforce your companies desired network topology at the same time. Be aware though, these Policy Definitions will only function if the Route Tables and Network Security Groups, referenced in your Azure Policy, are present in the Landing Zone that was provisioned earlier. 

## Putting Theory into Practice

Since this blog post is already covering a lot of topics, we have decided to create another blog post which will put all this theory into practice. Within this second blog post, we will explain the difficulties FOO, an insurance company, is facing regarding its network implementation and how the solutions presented above can help them overcome their challenges. 

Hope you enjoyed this new blog post.

[Bas van Bennekom](https://www.linkedin.com/in/bas-van-bennekom) and Stefan Stranger