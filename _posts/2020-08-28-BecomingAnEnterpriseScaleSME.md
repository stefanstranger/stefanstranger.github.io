---
layout: post
title: Becoming an Enterprise-Scale Subject Matter Expert
categories: [Azure, Microsoft, Enterprise-Scale]
tags: [Azure. Microsoft, Enterprise-Scale]
comments: true
---

If you are following me on <a href="https://twitter.com/sstranger" target="_blank">Twitter</a> you might have noticed that I got interested in <u>Microsoft's Enterprise-Scale</u>. And when I normally get interested in something new I want to learn more, and when you want to learn more you need to do research on the topic.

By setting myself a goal of becoming an Enterprise-Scale Subject Matter Expert (SME), I need to invest time to learn more about this topic. During this " journey" I will share my study notes and lessons learned. Normally I use <a href="https://onenote.com" target="_blank">OneNote</a> for my study notes but this time I'll use a series of blog posts. This way you and I can still review my study notes but at the same time you can help on improving the study notes. I really hope we can have some interesting interactions while becoming an Enterprise-Scale SME.

- [What is Enterprise-Scale? [3]](#what-is-enterprise-scale-3)
- [Becoming and Enterprise-Scale Subject Matter Expert](#becoming-and-enterprise-scale-subject-matter-expert)
- [References](#references)

## What is Enterprise-Scale? [3]

Enterprise-Scale is part of the <a href="https://azure.microsoft.com/en-us/cloud-adoption-framework/#:~:text=The%20Cloud%20Adoption%20Framework%20is%20proven%20guidance%20that%E2%80%99s,decision%20makers%20need%20to%20successfully%20achieve%20their%20" target="_blank">Cloud Adoption Framework</a> (CAF) [1], or more specifically the <u>Ready</u> phase of CAF [2].   

![Phase CAF](/assets/PhaseCAF.png)
  
<u>The Enterprise-Scale architecture provides prescriptive architecture guidance coupled with Azure best practices, and it follows design principles across the critical design areas for an organization's Azure environment and landing zones.</u> It is based on the following important 5 design principles:

* Subscription democratization
* Policy-driven governance
* Single control and management plane
* Application-centric and archetype neutral
* Align Azure-native design and roadmap

Furthermore, Enterprise-Scale within CAF lists many design guidelines, design considerations and recommendations. These 8 design areas can help you address the mismatch between and on-premises data center and cloud-design infrastructure. <u>It is not required that you implement all the design recommendations, as long as the chosen cloud-design infrastructure is aligned with the 5 design principles.</u>


The 8 design areas are as follows:

* Enterprise Agreement (EA) enrollment and Azure Active Directory tenants
* Identity and access management
* Management group and subscription organization
* Network topology and connectivity
* Management and monitoring
* Business continuity and disaster recovery
* Security, governance, and compliance
* Platform automation and DevOps

## Becoming and Enterprise-Scale Subject Matter Expert

After investigating Enterprise-Scale using both Microsoft internal and publicly available resources I got more and more excited about the advantages Enterprise-Scale could have for my customers.

By trying to become a Subject Matter Expert (SME) for Enterprise-Scale I want to challenge myself to learn as much as possible about Enterprise-Scale and in the end better help my customers during the Ready phase of their cloud adaption.

Before adoption can begin, you must create a landing zone to host the workloads that you plan to build or migrate to the cloud, and this where Enterprise-Scale comes in to play.

The first design principle I plan to further investigate is the <u>Policy-Driven Governance</u> design principle. The reason I want to start with this design principle is because this is a topic my current customer is interested in. Secondly it uses an interesting concept called <u>Compliance-as-Code</u> [5] which I had not heard about yet.

Enjoy my "study notes" while becoming an Enterprise-Scale SME.

If you also are interested to learn more about Enterprise-Scale please start reading the documentation shared in the Reference section below.

**Update 09-09-2020**:

[New blog post Enterpris-Scale Policy Driven Governance published](https://stefanstranger.github.io/2020/08/28/EnterpriseScalePolicyDrivenGovernance/)


## References

[1] <a href="https://docs.microsoft.com/en-us/azure/cloud-adoption-framework" target="_blank">Cloud Adoption Framework</a>

[2] <a href="https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/" target="_blank">Cloud Adaption Framework Ready Phase</a>

[3] <a href="https://techcommunity.microsoft.com/t5/azure-architecture-blog/enterprise-scale-for-azure-landing-zones/ba-p/1576575" target="_blank">Azure Architecture Blog -Enterprise-Scale for Azure landing zones</a>

[4] <a href="https://github.com/Azure/Enterprise-Scale" target="_blank">Enterprise-Scale information on Github</a>

[5] [PlatformOps in a Microsoft Enterprise-scale landing zone](https://www.linkedin.com/pulse/platformops-microsoft-enterprise-scale-landing-zone-anders-bonde/)