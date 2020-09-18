---
layout: post
title: Enterprise-Scale - Subscription Democratization
categories: [Azure]
tags: [Azure]
comments: true
---

In this third blog post in the blog series about becoming an Enterprise-Scale Subject Matter Expert I want to share what I did to better understand the Enterprise-Scale design principle <u>Subscription Democratization</u>.

For those who have not read my initial blog post [becoming an Enterprise-Scale Subject Matter Expert](https://stefanstranger.github.io/2020/08/28/BecomingAnEnterpriseScaleSME/) first and the second blog post about [Policy Driven Governance](https://stefanstranger.github.io/2020/08/28/EnterpriseScalePolicyDrivenGovernance/).

According to the Cloud Adoption Framework Subscription democratization is that <u>Subscriptions</u> should be used as a <u>unit of management and scale</u> aligned with business needs and priorities to support business areas and portfolio owners to accelerate application migrations and new application development. <u>Subscriptions should be provided to business units to support the design, development, and testing of new workloads and migration of workloads.</u>

# Documentation

* Start with enterprise scale - Subscription democratization [1]

* Management group and subscription organization - Subscription organization and governance [2]

# What does Subscription democratization mean?

When searching online I found [2] the following definition of data democratization:

"Data democratization can be defined as making digital information <u>accessible to the average non-technical user of information systems, without having to require the involvement of IT</u>."

According to Wikipedia [3] the Democratization of technology refers to the process by which access to technology rapidly continues to <u>become more accessible to more people</u>.

So if we follow above definitions the <u>Subscription Democratization</u> is the process by which <u>more people (DevOps or Application teams) have access to an Azure Subscription</u>, without having to require the involvement of IT.

So the idea is that a DevOps or Application team requests a Landing Zone (Azure Subscription) to the IT Platform team and when the Platform team has provisioned that Azure Subscription it is handed over the the DevOps or Application team.

# Subscription organization and governance
 
Subscriptions are a unit of management, billing, and scale within Azure. They play a critical role when you're designing for large-scale Azure adoption. This section helps you capture subscription requirements and design target subscriptions based on critical factors. These factors are environment type, ownership and governance model, organizational structure, and application portfolios. [4]

## Design considerations:

* Subscriptions serve as boundaries for assigning Azure policies. For example, secure workloads such as payment card industry (PCI) workloads typically require additional policies to achieve compliance. Instead of using a management group to group workloads that require PCI compliance, you can achieve the same isolation with a subscription. This way, you don't have too many management groups with a small number of subscriptions.
* 
* Subscriptions serve as a scale unit so that component workloads can scale within the platform subscription limits. Make sure to consider subscription resource limits during your workload design sessions.
* 
* Subscriptions provide a management boundary for governance and isolation, which creates a clear separation of concerns.
* 
* There's a manual process, planned future automation, that can be conducted to limit an Azure AD tenant to use only Enterprise Agreement enrollment subscriptions. This process prevents creation of Microsoft Developer Network subscriptions at the root management group scope.

## Design recommendations:

* Treat subscriptions as a democratized unit of management aligned with business needs and priorities.
    * Make subscription owners aware of their roles and responsibilities:
    * Perform an access review in Azure AD Privileged Identity Management quarterly or twice a year to ensure that privileges don't proliferate as users move within the customer organization.
* Take full ownership of budget spending and resource utilization.
* Ensure policy compliance and remediate when necessary.
* Use the following principles when identifying requirements for new subscriptions:
    * **Scale limits:** Subscriptions serve as a scale unit for component workloads to scale within platform subscription limits. For example, large, specialized workloads such as high-performance computing, IoT, and SAP are all better suited to use separate subscriptions to avoid limits (such as a limit of 50 Azure Data Factory integrations).
    * **Management boundary:** Subscriptions provide a management boundary for governance and isolation, which allows for a clear separation of concerns. For example, different environments such as development, test, and production are often isolated from a management perspective.
    * **Policy boundary:** Subscriptions serve as a boundary for the assignment of Azure policies. For example, secure workloads such as PCI typically require additional policies to achieve compliance. This additional overhead doesn't need to be considered holistically if a separate subscription is used. Similarly, development environments might have more relaxed policy requirements relative to production environments.
    * **Target network topology:** Virtual networks can't be shared across subscriptions, but they can connect with different technologies such as virtual network peering or Azure ExpressRoute. Consider which workloads must communicate with each other when you decide whether a new subscription is required.
* Group subscriptions together under management groups aligned within the management group structure and policy requirements at scale. Grouping ensures that subscriptions with the same set of policies and RBAC assignments can inherit them from a management group, which avoids duplicate assignments.
* Establish a dedicated management subscription in the Platform management group to support global management capabilities such as Azure Monitor Log Analytics workspaces and Azure Automation runbooks.
* Establish a dedicated identity subscription in the Platform management group to host Windows server Active Directory domain controllers, when necessary.
* Establish a dedicated connectivity subscription in the Platform management group to host an Azure Virtual WAN hub, private Domain Name System (DNS), ExpressRoute circuit, and other networking resources. A dedicated subscription ensures that all foundation network resources are billed together and isolated from other workloads.
* Avoid a rigid subscription model, and opt instead for a set of flexible criteria to group subscriptions across the organization. This flexibility ensures that as your organization's structure and workload composition changes, you can create new subscription groups instead of using a fixed set of existing subscriptions. One size doesn't fit all for subscriptions. What works for one business unit might not work for another. Some apps might coexist within the same landing zone subscription while others might require their own subscription.



# References

[1] [Subscription democratization](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/enterprise-scale/design-principles?branch#subscription-democratization)

[2] [What is Data Democratization? | Alation](https://www.alation.com/what-is-data-democratization/)

[3] [Democratization of technology | Wikipedia](https://en.wikipedia.org/wiki/Democratization_of_technology)

[4] [Subscription organization and governance](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/enterprise-scale/management-group-and-subscription-organization#subscription-organization-and-governance)