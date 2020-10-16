---
layout: post
title: Enterprise-Scale - Subscription Democratization
categories: [Azure]
tags: [Azure]
comments: true
---

Within this blog post, I would like to share the actions I took to better understand the Enterprise Scale principle of Subscription Democratization.

For those that have not read my initial blog post on "[Becoming an Enterprise-Scale Subject Matter Expert](https://stefanstranger.github.io/2020/08/28/BecomingAnEnterpriseScaleSME/)" and second blog post regarding "[Policy Driven Governance](https://stefanstranger.github.io/2020/08/28/EnterpriseScalePolicyDrivenGovernance/)", please do so as it provides a good introduction to the concept of Enterprise Scale Landing Zones (ESLZ).

I want to thank my colleague [Bas van Bennekom](https://www.linkedin.com/in/bas-van-bennekom) for all his valuable input while creating this blog post!

- [Documentation](#documentation)
- [What does Subscription democratization mean?](#what-does-subscription-democratization-mean)
  - [What are the benefits of Subscription Democratization?](#what-are-the-benefits-of-subscription-democratization)
  - [What are the design considerations?](#what-are-the-design-considerations)
  - [What are the design recommendations?](#what-are-the-design-recommendations)
- [What is a Landing Zone?](#what-is-a-landing-zone)
- [Putting theory into practice](#putting-theory-into-practice)
  - [Introduction](#introduction)
    - [Foo Insurance Company organization structure](#foo-insurance-company-organization-structure)
  - [Enterprise-Scale Reference implementation](#enterprise-scale-reference-implementation)
- [References](#references)

# Documentation

* Start with enterprise scale - Subscription democratization [1]

* Management group and subscription organization - Subscription organization and governance [2]

* What is an Azure Landing Zone? [3]

* Building workload specific Azure landing zones [4]



# What does Subscription democratization mean?

Subscription Democratization is one of the five guiding principles, besides Policy Driven Governance [5], Single Control and Management Plane, Application-centric and Archetype-neutral and Align Azure-native Design and Roadmap, used within ESLZ.

According to the Cloud Adoption Framework (CAF), this principle supports the use of Subscriptions as a unit of management and scale, aligned with business needs and priorities in order to accelerate application migrations and application development. In other words, Subscriptions should be provided to application teams to support the design, development and testing of new and existing workloads. Obviously, these Subscriptions should be coupled with policy and management boundaries, scale limits and a target networking topology. As such, these Subscriptions can be referred to as <u>Landing Zones</u> in which application teams can land their workloads in an accelerated fashion. Especially for large-scale adoption of Microsoft Azure, these Landing Zones play an important role by <u>enabling speed and agility while staying in control</u>.

## What are the benefits of Subscription Democratization?

By adopting the principle of Subscription Democratization, you will be able to:

* Remove the barriers of scale limits since Subscriptions become the primary and standard management entities for application teams. In other words, application teams do not hinder one another because of Subscription limitations.

* Enable application teams to migrate to a secure Landing Zone which provides a secure context based on the patterns of their applications.

* Enforce DevOps principles on your application teams as their responsibility encompasses the end-to-end management of their workloads.

* Enable awareness regarding cost management as application teams can review their consumption and proactively take actions (e.g. Azure Advisor and Billing) to reduce unnecessary consumption of Azure Services.

## What are the design considerations?

* Subscriptions serve as boundaries for assigning Azure policies. For example, secure workloads such as payment card industry (PCI) workloads typically require additional policies to achieve compliance. Instead of using a management group to group workloads that require PCI compliance, you can achieve the same isolation with a subscription. This way, you don't have too many management groups with a small number of subscriptions.

* Subscriptions serve as a scale unit so that component workloads can scale within the platform subscription limits. Make sure to consider subscription resource limits during your workload design sessions.

* Subscriptions provide a management boundary for governance and isolation, which creates a clear separation of concerns.

* There's a manual process, planned future automation, that can be conducted to limit an Azure AD tenant to use only Enterprise Agreement enrollment subscriptions. This process prevents creation of Microsoft Developer Network subscriptions at the root management group scope.

## What are the design recommendations?

* Treat subscriptions as a democratized unit of management aligned with business needs and priorities.
    * Make subscription owners aware of their roles and responsibilities:
    * Perform an access review in Azure AD Privileged Identity Management quarterly or twice a year to ensure that privileges don't proliferate as users move within the customer organization.
    
* Take full ownership of budget spending and resource utilization.

* Ensure policy compliance and remediate when necessary.

* Use the following principles when identifying requirements for new subscriptions:
    * **Scale limits:**   
Subscriptions serve as a scale unit for component workloads to scale within platform subscription limits. For example, large, specialized workloads such as high-performance computing, IoT, and SAP are all better suited to use separate subscriptions to avoid limits (such as a limit of 50 Azure Data Factory integrations).

    * **Management boundary:**   
Subscriptions provide a management boundary for governance and isolation, which allows for a clear separation of concerns. For example, different environments such as development, test, and production are often isolated from a management perspective.

    * **Policy boundary:**   
Subscriptions serve as a boundary for the assignment of Azure policies. For example, secure workloads such as PCI typically require additional policies to achieve compliance. This additional overhead doesn't need to be considered holistically if a separate subscription is used. Similarly, development environments might have more relaxed policy requirements relative to production environments.

    * **Target network topology:**   
Virtual networks can't be shared across subscriptions, but they can connect with different technologies such as virtual network peering or Azure ExpressRoute. Consider which workloads must communicate with each other when you decide whether a new subscription is required.

* Group subscriptions together under management groups aligned within the management group structure and policy requirements at scale. Grouping ensures that subscriptions with the same set of policies and RBAC assignments can inherit them from a management group, which avoids duplicate assignments.

* Establish a dedicated management subscription in the Platform management group to support global management capabilities such as Azure Monitor Log Analytics workspaces and Azure Automation runbooks.

* Establish a dedicated identity subscription in the Platform management group to host Windows server Active Directory domain controllers, when necessary.

* Establish a dedicated connectivity subscription in the Platform management group to host an Azure Virtual WAN hub, private Domain Name System (DNS), ExpressRoute circuit, and other networking resources. A dedicated subscription ensures that all foundation network resources are billed together and isolated from other workloads.

* Avoid a rigid subscription model, and opt instead for a set of flexible criteria to group subscriptions across the organization. This flexibility ensures that as your organization's structure and workload composition changes, you can create new subscription groups instead of using a fixed set of existing subscriptions. One size doesn't fit all for subscriptions. What works for one business unit might not work for another. Some apps might coexist within the same landing zone subscription while others might require their own subscription.

# What is a Landing Zone?

As mentioned earlier, Subscription Democratization supports the use of a Subscription as the primary and standard management entity for application teams. A Landing Zone thus uses a Subscription as a logical container for grouping resources but also accounts for items related to security, governance, networking, and identity. For instance, Landing Zones are associated with a set of Azure Policies that provide the guardrails for application teams to operate in. Moreover, networking components such as Virtual Networks and Route Tables are provided out of the box. In other words, Landing Zones encompass all platform components required to support the application portfolio of a customer.

However, to satisfy the requirements of different application teams, it is recommended to develop different types of Landing Zones. These Landing Zones should align with the Application Archetypes of teams that want to migrate to Microsoft Azure. Put differently, Landing Zones of type ‘Hybrid’ should be able to satisfy the security, governance, networking, and identity requirements of applications that need hybrid connectivity. Hence, the <u>Application Archetypes</u> determine the number of Landing Zone types that need to be developed since every one of them is optimized for different requirements. For instance, the security requirements for a Landing Zone of type ‘Development’ are less strict in comparison to a Landing Zone of type ‘Production’. Consequently, the Azure Policies associated with both Landing Zones are likely to be different. For that reason, it is important to group applications with similar requirements into Application Archetypes. Subsequently, these archetypes are translated into different types of Landing Zones under which the Subscriptions of different application teams are grouped.
[4]

# Putting theory into practice

In the following use case, we follow the practices of Subscription Democratization for the imaginary customer FOO. This should help to provide a clearer overview of concepts such as Application Archetypes and Landing Zones.

## Introduction

**FOO - an insurance company**

FOO is going through a transformation to become more Agile and to embrace DevOps practices throughout the whole organization in order to reduce the time-to-market of its products and stay ahead of its competitors.

The IT department of FOO plays an important role in this transformation as it needs to enable application teams instead of blocking their progress. For that reason, they have started to hand out environments in Microsoft Azure in which application teams can experiment with public cloud. Unfortunately, the IT department is not able to unlock the benefits of public cloud as their existing way of working has not changed. Consequently, application teams must wait a long time for their environments as every workload needs to be evaluated in terms of security and service management first.

For that reason, the CIO of FOO has decided to follow the guidance of Microsoft and adopt the concepts of ESLZ to resolve these issues. Before we dive deeper into the implementation of ESLZ in FOO, some background information is provided.

### Foo Insurance Company organization structure

![](/assets/foo_organization_structure.png)

*Figure 1: Organizational Structure*

One of the application teams in the Claims department is responsible for developing an application (MyClaims) that allows customers to submit insurance claims using any device (e.g. mobile). These claims can be filed by uploading pictures and providing a general description of the sustained damage. For FOO, this will be the first workload that is onboarded to Microsoft Azure while adopting the ESLZ concept.

![MyClaims Solution Design](/assets/myclaims_solution_design.png)

*Figure 2: Solution Design*

As you can see in Figure 2, the application team responsible for developing the MyClaims application has created an initial solution design. This design adopts a three-tier architecture, encompassing a client layer (e.g. Web App), business layer (e.g. Function App) and data layer (e.g. SQL Database) as it offers advantages in terms of scalability, performance and data integrity. Since customers should be able to create claims, it needs to be publicly accessible via the internet. To mitigate common exploits and vulnerabilities, the application team decided to use Azure Front Door as well.

With a solution design in place, the application team is ready to land on Microsoft Azure. However, the IT department does not know where their environment should be deployed when following ESLZ guidance. For that reason, we will help them a little!

## Enterprise-Scale Reference implementation

The IT department of FOO has selected the Wingtip reference implementation to conduct a pilot on the concepts of ESLZ. This pilot should validate whether ESLZ truly reduces time-to-market and thus enables FOO to stay ahead of its competitors.

![Wingtip Azure Architecture overview](/assets/EnterpriseScaleArchitecture.png)

*Figure 3: Wingtip Reference Implementation*

Figure 3 provides an overview on the management structure of the Wingtip reference implementation. The ‘ES-Management’ Management Group hosts the ‘Management’ Subscription which will be used for management purposes such as monitoring and alerting. For that reason, Azure Services such as Log Analytics and Automation Account are deployed in this Subscription out-of-the-box.

Next to that, the ‘ES-Online’ Management Group has been deployed. This Management Group will include Landing Zones (Subscriptions) that host applications requiring internet connectivity. Azure Policies, capable of mitigating all the risks associated with this Application Archetype, are assigned to that Management Group. Since the MyClaims application has similar requirements (e.g. internet connectivity), the IT department of FOO creates a new Landing Zone for the application team in which they can start building their solution. This resulted in the Landing Zone Structure visualized in Figure 4.

![My Claims Landing Zone Architecture](/assets/MyClaimsEnterpriseScaleArchitecture.png)

*Figure 4: Landing Zone Architecture*

To test whether you fully understand the concepts of Subscription Democratization, Landing Zones and Application Archetypes, let’s consider a different use case.

Another application team also want to host their application on Microsoft Azure. This application is for internal use only and requires hybrid connectivity to the on-premises datacenter of FOO. The IT department of FOO struggles with the following questions:

* Should FOO deploy a new Management Group under the 'ES-Landingzones' Management Group, or can they use the 'ES-Online' Management Group?

* Should FOO use the existing Azure Policies applied on the 'ES-Online' Management Group or should they first determine the risks associated with applications that require hybrid connectivity?

* Should FOO grant this new application team a Subscription or Resource Group for hosting their resources?

Can you help them answer these questions?

# References

[1] [Subscription democratization](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/enterprise-scale/design-principles?branch#subscription-democratization)

[2] [Subscription organization and governance](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/enterprise-scale/management-group-and-subscription-organization#subscription-organization-and-governance)

[3] [Cloud Adoption Framework - What is an Azure Landing Zone](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone/)

[4] [Building workload specific Azure landing zones](https://techcommunity.microsoft.com/t5/azure-architecture-blog/building-workload-specific-azure-landing-zones/ba-p/1677941)

[5] [Blog post Stefan Stranger - Enterprise-Scale Policy Driven Governance](https://stefanstranger.github.io/2020/08/28/EnterpriseScalePolicyDrivenGovernance/)