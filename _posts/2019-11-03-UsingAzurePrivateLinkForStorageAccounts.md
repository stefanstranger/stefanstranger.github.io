---
layout: post
title: Using Azure Private Link (Preview) for Storage Accounts
categories: [Azure]
tags: [Azure]
comments: true
---


I recently started to investigate **Azure Private Link** to access Azure Storage (Azure PaaS Service) over a Private Endpoint in our virtual network.

Azure Private Link provides a number of benefits (see below for more information) but the main reason I was investing Private Links was to not expose the Storage Account to the public Internet.

| Important                                |
|------------------------------------------|
| This public preview is provided without a service level agreement and should not be used for production workloads. Certain features may not be supported, may have constrained capabilities, or may not be available in all Azure locations. <a href="https://azure.microsoft.com/support/legal/preview-supplemental-terms/" target="_blank">See the Supplemental Terms of Use for Microsoft Azure Previews</a> for details. For known limitations, see <a href="https://docs.microsoft.com/en-us/azure/private-link/private-endpoint-overview#limitations" target="_blank">Private Endpoint</a> and <a href="https://docs.microsoft.com/en-us/azure/private-link/private-link-service-overview#limitations" target="_blank">Private Link Service</a>.<br><br>

Azure Private Link enables you to access Azure PaaS Services (for example, Azure Storage and SQL Database) and Azure hosted customer/partner services over a Private Endpoint in your virtual network. **Traffic between your virtual network and the service traverses over the Microsoft backbone network, eliminating exposure from the public Internet**. You can also create your own Private Link Service in your virtual network (VNet) and deliver it privately to your customers. The setup and consumption experience using Azure Private Link is consistent across Azure PaaS, customer-owned, and shared partner services.

## Key benefits (from <a href="https://docs.microsoft.com/en-us/azure/private-link/private-link-overview#key-benefits" target="_blank">Microsoft docs</a>)

Azure Private Link provides the following benefits:

* **Privately access services on the Azure platform:** Connect your virtual network to services running in Azure privately without needing a public IP address at the source or destination. Service providers can render their services privately in their own virtual network and consumers can access those services privately in their local virtual network. The Private Link platform will handle the connectivity between the consumer and services over the Azure backbone network.

* **On-premises and peered networks:** Access services running in Azure from on-premises over ExpressRoute private peering/VPN tunnels (from on-premises) and peered virtual networks using private endpoints. There is no need to set up public peering or traverse the internet to reach the service. This ability provides a secure way to migrate workloads to Azure.

* **Protection against data exfiltration:** With Azure Private Link, the private endpoint in the VNet is mapped to a specific instance of the customer's PaaS resource as opposed to the entire service. Using the private endpoint consumers can only connect to the specific resource and not to any other resource in the service. This in built mechanism provides protection against data exfiltration risks.

* **Global reach:** Connect privately to services running in other regions. This means that the consumer's virtual network could be in region A and it can connect to services behind Private Link in region B.

* **Extend to your own services:** Leverage the same experience and functionality to render your own service privately to your consumers in Azure. By placing your service behind a Standard Load Balancer you can enable it for Private Link. The consumer can then connect directly to your service using a Private Endpoint in their own VNet. You can manage these connection requests using a simple approval call flow. Azure Private Link works for consumers and services belonging to different Active Directory tenants as well.

## Demo Environment Infrastructure

To investigate the different scenario's for the use-cases of Private Endpoints I setup the following demo environment. The Azure Private Link to access Azure Storage needed to be accessible from both Azure Resources as from on-premises resources.

![](/assets/27-10-2019-3.png)

<img src="/assets/27-10-2019-3-1.png" width="25">Notes

**Resources**

* (Blob) Storage Account with Private Endpoint 
* Virtual Machine to test connecting to the Storage Account via the Private Endpoint.

**Virtual Network**

Shared Virtual Network (pl-shared-vnet) for both the Virtual Machine(s) and the Private Endpoint for the Storage Account.

| Property | Value | Comment |
|----------|----------|---|
| Address space | 10.2.0.0/27 |  |
| Subnets | Default (10.2.0.0/27) | |
| DNS Servers | Custom (10.2.16.4) | IP Address of DNS Server with Conditional Forwarders for privatelink.blob.core.net |
| Peerings | Peer with pl-dns-vnet | Peered with the Vnet where the DNS Server with Conditional  Forwarders is located |
| Private endpoints | Name: pl-01-sa-blob-pep | |

<img src="/assets/27-10-2019-3-2.png" width="25">Notes

**Resources**

* Private DNS Zone for azure.contoso.com
* Private DNS Zone for privatelink.blob.core.windows.net

**privatelink.blob.core.windows.net Private DNS Zone**

| Property | Value | Comment |
|----------|----------|---|
| Virtual Network Links | DNS Vnet/Shared VNet | Links to both the DNS Vnet where the DNS Server with Conditional Forwarders is located and the Shared VNet where the Private Endpoint and (test) VM are located |

![](/assets/27-10-2019-4.png)

A-record for Blob Storage Account Private Link screenshot.

![](/assets/27-10-2019-4.1.png)

**azure.contoso.com Private DNS Zone**

This Private DNS zone is created to auto register A-record for Virtual Machines for the azure.contoso.com Private DNS zone. Virtual Machines register to the Azure Private DNS Zone for name resolution across Virtual Networks.

![](/assets/27-10-2019-4.2.png)

| Property | Value | Comment |
|----------|----------|---|
| Virtual Network Links | Shared VNet | Links to the Shared VNet where the Private Endpoint and (test) VM are located. Auto registration is enabled |

<table border="0" align="left">
  <tr>
	<th><img src="/assets/27-10-2019-3-3.png" width="25"></th>
	<th>Notes</th>
  </tr>
</table>

**Resources**

This Resource Group contains a DNS Server with Conditional Forwarders for azure.contoso.com and privatelink.blob.core.windows.net within a Vnet linked to the Shared VNet.

* Windows Virtual Machine with DNS role installed
* Virtual Network with Subnet

**DNS Server Configuration**

![](/assets/27-10-2019-5.png)

IP address 168.63.129.16 is a virtual public IP address that is used to facilitate a communication channel to Azure platform resources.

No forwarders configured. Root hints are used.

**Virtual Network**

| Property | Value | Comment |
|----------|----------|---|
| Address space | 10.2.16.0/27 |  |
| Subnets | DNSSubnet (10.2.16.0/29) | |
| DNS Servers | Custom (10.2.16.4) | IP Address of DNS Server with Conditional Forwarders for privatelink.blob.core.net |
| Peerings | Peer with pl-shared-vnet | Peered with the Vnet where the VMs and Private Endpoint is l

<img src="/assets/27-10-2019-3-4.png" width="25"> Notes

To enable access to the private endpoint for the Storage Account from on-premises servers a conditional forwarder needs to be configured on the on-premises DNS server for privatelink.blob.core.windows.net and azure.contoso.com to the DNS Server in Azure.

## <a href="https://docs.microsoft.com/en-us/azure/private-link/private-endpoint-overview#dns-configuration" target="_blank">DNS Configuration</a>

When connecting to a private link resource using a fully qualified domain name (FQDN) as part of the connection string, it's important to correctly configure your DNS settings to resolve to the allocated private IP address

The following options are available to configure the DNS settings for private endpoints:

* Host file (only for testing)
* Private DNS Zone
* Custom DNS Server

For Azure services, use the recommended zone names as described in the following table:


| Private Link resource type               | Subresource                              | Zone name                           |
|------------------------------------------|------------------------------------------|-------------------------------------|
| SQL DB/DW (Microsoft.Sql/servers)        | Sql Server (sqlServer)                   | privatelink.database.windows.net    |
| Storage Account (Microsoft.Storage/storageAccounts) | Blob (blob, blob_secondary)              | privatelink.blob.core.windows.net   |
| Storage Account (Microsoft.Storage/storageAccounts) | Table (table, table_secondary)           |  privatelink.table.core.windows.net |
| Storage Account (Microsoft.Storage/storageAccounts) | Queue (queue, queue_secondary)           | privatelink.queue.core.windows.net  |
| Storage Account (Microsoft.Storage/storageAccounts) | File (file, file_secondary)              | privatelink.file.core.windows.net   |
| Storage Account (Microsoft.Storage/storageAccounts) | Web (web, web_secondary)                 |  privatelink.web.core.windows.net   |
| Data Lake File System Gen2 (Microsoft.Storage/storageAccounts) | Data Lake File System Gen2 (dfs, dfs_secondary) | privatelink.dfs.core.windows.net    |

### Create Private Endpoint for Storage Account

Follow the steps to create the Private Endpoint for Storage Account explained <a href="https://docs.microsoft.com/en-us/azure/private-link/create-private-endpoint-storage-portal#create-your-private-endpoint" target="_blank">here</a>.

### Azure Private DNS Zone

Private Endpoints allow integration with private DNS zones. If you don't integrate your endpoint with a DNS zone, you will need to create records on either your own DNS server or via host file updates on each machine (not recommended).

Screenshots Private DNS Zone for blob Storage Account
![](/assets/27-10-2019-1.png)

![](/assets/27-10-2019-2.png)

## Scenario's

### Resolve DNS Name (nslookup) from host in same Virtual Network as storage account

```PowerShell
$env:computername
Get-NetIPConfiguration
Resolve-DnsName pl01sa.blob.core.windows.net
Resolve-DnsName www.stranger.nl
```

![](/assets/nslookup1.gif)

IP address for pl01sa.blob.core.windows.net is 10.2.0.5

We can now try to access the Blob Storage Account using the Azure Storage Explorer to validate access.

![](/assets/27-10-2019-6.png)

As shown above we have access to the blob storage account container from the Virtual Machine in the same Virtual Network as the Private Link Storage Account.

### Resolve DNS Name (nslookup) from host on internet

![](/assets/nslookup2.gif)

IP address for pl01sa.blob.core.windows.net is 13.95.96.176

From internet we should not have access to the blob storage account container if everything is correctly configured.

![](/assets/27-10-2019-7.png)

## Lessons learned

While investigating Azure Private Link I learned some lessons I would like to share.

### Private Endpoints and Service Endpoints

Private Endpoints **cannot be deployed** on subnets enabled for service endpoints or subnets delegated to specialized workloads.

See <a href="https://docs.microsoft.com/en-us/azure/private-link/private-endpoint-overview#limitations" target="_blank">Private Endpoint limitations</a> for more information.

Virtual Network (VNet) **service endpoints** extend your virtual network private address space and the identity of your VNet to the Azure services, over a direct connection. Endpoints allow you to secure your critical Azure service resources to only your virtual networks. Traffic from your VNet to the Azure service always remains on the Microsoft Azure backbone network.

### Azure DNS Private Zones and Conditional forwarding

At this time, conditional forwarding is not supported (could be needed for enabling DNS resolution between Azure and on-premises networks)

### Network Security Groups

Network Security Groups (NSG) don't work on Private Endpoints will still work on Vnet Subnets. The rules will not be effective on traffic processed by the private endpoint. You must have network policies enforcement disabled to deploy private endpoints in a subnet. NSG is still enforced on other workloads hosted on the same subnet. Routes on any client subnet will be using an /32 prefix, changing the default routing behavior requires a similar UDR.

**References:**
* <a href="https://docs.microsoft.com/en-us/azure/private-link/private-link-overview" target="_blank">What is Azure Private Link? (Preview)</a>
* <a href="https://docs.microsoft.com/en-us/azure/private-link/create-private-endpoint-storage-portal" target="_blank">Connect privately to a storage account using Azure Private Endpoint</a>
* <a href="https://docs.microsoft.com/en-us/azure/sql-database/sql-database-private-endpoint-overview" target="_blank">Private Link for Azure SQL Database and Data Warehouse (Preview)</a>
* <a href="https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview" target="_blank">Virtual Network Service Endpoints Overview</a>
* <a href="https://docs.microsoft.com/en-us/azure/dns/dns-domain-delegation" target="_blank">Delegation with Azure DNS</a>
* <a href="https://docs.microsoft.com/en-us/azure/virtual-network/what-is-ip-address-168-63-129-16" target="_blank">What is the IP Address 168.63.129.16?</a>
* <a href="https://github.com/Azure/azure-quickstart-templates/tree/master/301-dns-forwarder/" target="_blank">Azure Quick Start template for dns forwarder</a>
* <a href="https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances#name-resolution-using-your-own-dns-server" target="_blank">Name resolution for resources in Azure virtual networks</a>