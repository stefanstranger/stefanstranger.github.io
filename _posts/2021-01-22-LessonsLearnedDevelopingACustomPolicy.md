---
layout: post
title: Lessons learned while developing a custom Azure Policy
categories: [Azure]
tags: [Azure, Azure Policy, Governance, Development]
comments: true
---


I learned something new on an issue while trying to create a Custom Azure Policy to audit if an Azure Eventhub has a Private Link configured.

Azure Private Link Service enables you to access Azure Services (for example, *Azure Event Hubs*, Azure Storage, and Azure Cosmos DB) and Azure hosted customer/partner services over a private endpoint in your virtual network.

A private endpoint is a network interface that connects you privately and securely to a service powered by Azure Private Link. The private endpoint uses a private IP address from your virtual network, effectively bringing the service into your virtual network.

Before discussing the encountered issue let's first go through the steps I followed to develop this custom Azure Policy.

- [Azure Policy Development steps](#azure-policy-development-steps)
  - [Deploy Azure Event Hub](#deploy-azure-event-hub)
  - [Search for Azure built-in Policies for Event Hub](#search-for-azure-built-in-policies-for-event-hub)
  - [Search for Event Hub Policy aliases](#search-for-event-hub-policy-aliases)
  - [Create custom Policy Definition](#create-custom-policy-definition)
  - [Assign Azure Policy](#assign-azure-policy)
  - [Trigger on-demand Azure Policy compliance evaluation](#trigger-on-demand-azure-policy-compliance-evaluation)
  - [Validate non-compliancy](#validate-non-compliancy)
- [Issue](#issue)
- [Reference](#reference)

## Azure Policy Development steps

I normally follow the following steps:

1. Deploy Azure Resource with required configuration

2. Search for Azure Built-in Policies as a starting point

3. Search for Azure Event Hub aliases

4. Create custom Policy Definition

5. Deploy Policy Definition

6. Assign Policy Definition

7. Trigger on-demand Azure Policy compliance evaluation

### Deploy Azure Event Hub

Just follow the steps documented on [Microsoft Docs](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-create) to deploy your Azure Event Hub. Make sure you select the standard pricing tier otherwise you won't be able to create a Private endpoint connection later on.

You should now have deployed the Event Hub and as a next step you need to create the Private endpoint connection. Again there are multiple ways to do so just choose the option you are most comfortable with using the [Microsoft Docs documentation](https://docs.microsoft.com/en-us/azure/event-hubs/private-link-service).

You should now have something similar deployed as is being showed below.

![Event Hub configuration screenshot](/assets/22-01-2021-01.png)

With a private endpoint configured in the Network configuration.

![Private endpoint configuration screenshot](/assets/22-01-2021-02.png)

### Search for Azure built-in Policies for Event Hub

Why reinvent the wheel if there might be some existing Event Hub Azure Policies? Start searching the built-in Azure Policies first, using the Azure Portal and with some filters.

![Event Hub built-in Azure Event Hub Policies screenshot](/assets/22-01-2021-03.png)

None of the above Azure Policy Definitions seems to evaluate against a Private link connection for Event Hub.

Let's change the search criteria a bit.

![All Categories overview of built-in Policies for Private Endpoint](/assets/22-01-2021-04.png)

Open one of the above Policy Definitions to see if we can use this is a starting point for our custom Azure Policy to audit if a private endpoint is enabled for Azure Event Hub.

![Private endpoint should be enabled for PostSQL Servers Policy Definition screenshot](/assets/22-01-2021-05.png)

The interesting part in this Policy Definition configuration is the **existenceCondition**.

The effect of the Policy is AuditIfNotExists. AuditIfNotExists enables auditing of resources **related** to the resource that matches the if condition, but don't have the properties specified in the details of the then condition.

So in this example if the if condition Resource of type "Microsoft.DBforPostgreSQL/servers" is met we can audit resource related to that resource being a Private endpoint connection.

**AuditIfNotExists** runs after a Resource Provider has handled a create or update resource request and has returned a success status code. The audit occurs if there are no related resources or if the resources defined by ExistenceCondition don't evaluate to true.

When a Resource type "Microsoft.DBforPostgreSQL/servers" is found in the AuditIfNotExists then condition, an existence condition is executed against Resource Type "Microsoft.DBforPostgreSQL/servers/privateEndpointConnections" for a field "Microsoft.DBforPostgreSQL/servers/privateEndpointConnections/privateLinkServiceConnectionState.status" with a value equal to "Approved"

In an flow chart this looks as follows.

![Flow chart over of PostSQL Servers Policy Definition](/assets/22-01-2021-06.png)

This seems like a good starting point for creating the new custom Azure Policy.

### Search for Event Hub Policy aliases

Aliases in resource policies enable you to restrict what values or conditions are permitted for a property on a resource.

Using Azure PowerShell we can search for Private Endpoint connection aliases for Eventhub:

```PowerShell
# Install Graphical tools for easy searching
Install-Module Microsoft.PowerShell.GraphicalTools
Install-Module Microsoft.PowerShell.ConsoleGuiTools
# Retrieve Azure Policy aliases
Get-AzPolicyAlias -NamespaceMatch 'eventhub' | Out-ConsoleGridView -OutputMode Multiple | Select-Object -ExcludeProperty Aliases
```

![Animated gif](/assets/22-01-2021-07.gif)

Based on the output from above alias query we should be able to use the following alias:

"Microsoft.EventHub/namespaces/privateEndpointConnections/privateLinkServiceConnectionState.status" in our custom Private endpoint should be enabled for Event Hub Policy Definition.

### Create custom Policy Definition

Follow the steps decribed at [Microsoft Docs - Tutorial: Create a custom policy definition](https://docs.microsoft.com/en-us/azure/governance/policy/tutorials/create-custom-policy-definition).

When you reached the compose the definition step use the following definition:

```json
{
  "properties": {
    "displayName": "Private endpoint should be enabled for Event Hub",
    "policyType": "Custom",
    "mode": "All",
    "description": "Private endpoint connections enforce secure communication by enabling private connectivity to Azure Event Hub. Configure a private endpoint connection to enable access to traffic coming only from known networks and prevent access from all other IP addresses, including within Azure.",
    "parameters": {
      "effect": {
        "type": "String",
        "metadata": {
          "displayName": "Allowed Azure Policy Effect",
          "description": "The allowed effect for this Azure Policy"
        },
        "defaultValue": "AuditIfNotExists"
      }
    },
    "policyRule": {
      "if": {
        "field": "type",
        "equals": "Microsoft.EventHub/Namespaces"
      },
      "then": {
        "effect": "[parameters('effect')]",
        "details": {
          "type": "Microsoft.EventHub/namespaces/privateEndpointConnections",
          "existenceCondition": {
            "field": "Microsoft.EventHub/namespaces/privateEndpointConnections/privateLinkServiceConnectionState.status",
            "equals": "Approved"
          }
        }
      }
    }
  }
}
```

You can copy and past the following code from above Policy Definition in the Azure Portal where you can do a first test of the Policy Definition deployment.

```json
{
  "mode": "All",
  "parameters": {
    "effect": {
      "type": "String",
      "metadata": {
        "displayName": "Allowed Azure Policy Effect",
        "description": "The allowed effect for this Azure Policy"
      },
      "defaultValue": "AuditIfNotExists"
    }
  },
  "policyRule": {
    "if": {
      "field": "type",
      "equals": "Microsoft.EventHub/Namespaces"
    },
    "then": {
      "effect": "[parameters('effect')]",
      "details": {
        "type": "Microsoft.EventHub/namespaces/privateEndpointConnections",
        "existenceCondition": {
          "field": "Microsoft.EventHub/namespaces/privateEndpointConnections/privateLinkServiceConnectionState.status",
          "equals": "Approved"
        }
      }
    }
  }
}
```

![Screenshot of Policy Definition in Azure Portal](/assets/22-01-2021-08.png)

And save the Policy to deploy the Policy via the Azure Portal.

### Assign Azure Policy

After deploying the "Private endpoint should be enabled for Event Hub" custom Azure Policy we need to assign the Azure Policy at the correct scope.

You can follow the documentation at [Microsoft Docs called Quickstart: Create a policy assignment to identify non-compliant resources](https://docs.microsoft.com/en-us/azure/governance/policy/assign-policy-portal) to assign the previously deployed custom Azure Policy.

In our example we are assigning the custom Azure Policy at the highest scope in our Enterprise-Scale environment at Management Group ES scope.

![Screenshot of Policy Assignment in Azure Portal](/assets/22-01-2021-09.png)

![Screenshot of Policy Assignment in Azure Portal](/assets/22-01-2021-10.png)

### Trigger on-demand Azure Policy compliance evaluation

So the custom Azure Policy Definition is assigned, let's now trigger a Policy compliance evaluation to see if the earlier deployed Event Hub with Private link connection is compliant.

If found that using the Azure CLI is the easiest way to trigger an on-demand Azure Policy compliance evaluation. Especially because we don't want to wait on the Azure platform to trigger that evaluation, because that can take some time (30 mins) to be triggered.

Run the following Azure CLI commands.

```PowerShell
az policy state trigger-scan --resource-group "es-demo-rg"
```

Keep in mind this can take a couple of mins to finish. So get yourself a cup of tea or coffee ☕ Below trigger run for 8 minutes.

![Animated gif showing the Azure CLI trigger Policy scan](/assets/22-01-2021-11.gif)

You can check in the Azure Portal the status of the Policy.

![Screenshot of Azure Policy Compliance state in Azure Portal](/assets/22-01-2021-12.png)

### Validate non-compliancy

The final test if the custom Azure Policy works as expected is by removing the Private link connection from the earlier deployed Event Hub.

![Screenshot of Event Hub Private link connection removal in Azure Portal](/assets/22-01-2021-13.png)

Run the on-demand Azure Policy compliance evaluation Azure CLI command again after you have removed the Private link connection and look at the result.

Run again the following Azure CLI commands.

```PowerShell
az policy state trigger-scan --resource-group "es-demo-rg"
```

We now see that the Event Hub is non-compliant because there is no Private link connection configured.

![Screenshot of non-compliant Event Hub in Azure Portal](/assets/22-01-2021-14.png)

## Issue

So what was the issue where it all started with? When I first started developing this custom Azure Policy I started with the following Policy Definition. 

```json
{
    "properties": {
      "displayName": "Not working Private endpoint should be enabled for Event Hub",
      "policyType": "Custom",
      "mode": "All",
      "description": "Private endpoint connections enforce secure communication by enabling private connectivity to Azure Event Hub. Configure a private endpoint connection to enable access to traffic coming only from known networks and prevent access from all other IP addresses, including within Azure.",
      "parameters": {
        "effect": {
          "type": "String",
          "metadata": {
            "displayName": "Allowed Azure Policy Effect",
            "description": "The allowed effect for this Azure Policy"
          },
          "defaultValue": "AuditIfNotExists"
        }
      },
      "policyRule": {
        "if": {
          "allOf": [
            {
              "field": "type",
              "equals": "Microsoft.EventHub/Namespaces"
            },
            {
              "field": "Microsoft.EventHub/namespaces/privateEndpointConnections/privateLinkServiceConnectionState.status",
              "notLike": "Approved"
            }
          ]
        },
        "then": {
          "effect": "audit"
        }
      }
    }
 }
```

Do you see the difference between the working version and this version?

Let's compare the Policy Rule side-by-side:

![Table comparing working and non working custom Policies](/assets/22-01-2021-15.png)

**So why does not the right audit Azure Policy works?**

The reason the audit policy does not work is because it is auditing a resource of type namespaces, but attempting to apply evaluation criteria to a *different resource type*, namespaces/privateEndpointConnections. Evaluating the subtype requires another GET request after GETting the parent namespaces resource, and this is not performant enough.

You can validate above using the Azure Resource Explorer.

![Screenshot over Azure Resource Explorer showing different resource types](/assets/22-01-2021-16.png)

This exact problem is why [DeployIfNotExists](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/effects#deployifnotexists) (DINE) and [AuditIfNotExists](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/effects#auditifnotexists) (AINE) are provided: it allows the policy writer to specifically evaluate only the subtypes and fields they need to apply conditions to, and either audit or update the resource asynchronously. Doing this much processing in a deny or audit policy would cause serious performance problems in ARM, since such evaluations must be done in the PUT path and can’t be done asynchronously.

The key to understanding this is to realize that the if {} section of a policy rule evaluates **only one resource type at a time**.

Compare this with the following custom Azure Policy to audit the deployment of an Azure Container Registry if the Admin user setting is set to Enabled.

```json
"policyRule": {
      "if": {
        "allOf": [
          {
            "field": "type",
            "equals": "Microsoft.ContainerRegistry/registries"
          },
          {
            "field": "Microsoft.ContainerRegistry/registries/adminUserEnabled",
            "equals": "True"
          }
        ]
      },
      "then": {
        "effect": "[parameters('effect')]"
      }
    }
```

![Screenshot Resource Explorer for Microsoft.ContainerRegistry](/assets/22-01-2021-17.png)

In this case there is no different resource type to be evaluated.

I want to thank [Chris Stackhouse](https://www.linkedin.com/in/chris-stackhouse-dev/) helping me understand the logic of the existenceCondition in Azure Policies.

Hope you also learned something new.

## Reference

- [Microsoft Docs - Allow access to Azure Event Hubs namespaces via private endpoints](https://docs.microsoft.com/en-us/azure/event-hubs/private-link-service)

- [Microsoft Docs - Quickstart: Create an event hub using Azure portal](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-create)

- [Microsoft Docs - Understand Azure Policy effects](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/effects)

- [Blog Post - More resource policy aliases](https://azure.microsoft.com/en-us/blog/more-resource-policy-aliases/)

- [Github Azure Policy Samples repository](https://github.com/Azure/azure-policy)

- [Microsoft Docs - Tutorial: Create a custom policy definition](https://docs.microsoft.com/en-us/azure/governance/policy/tutorials/create-custom-policy-definition)

- [Daniel's Tech Blog - Trigger an on-demand Azure Policy compliance evaluation scan](https://www.danielstechblog.io/trigger-an-on-demand-azure-policy-compliance-evaluation-scan)