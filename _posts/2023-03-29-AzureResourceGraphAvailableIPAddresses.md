---
layout: post
title: Retrieve available IP addresses from Azure Resource Graph
categories: [Azure, Azure Resource Graph]
tags: [Azure, Azure Resource Graph]
comments: true
---

- [How to calculate the number of IP addresses?](#how-to-calculate-the-number-of-ip-addresses)
- [Azure Resource Graph](#azure-resource-graph)

Hi friends, I want to share with you a pretty cool Kusto Azure Resource Graph query that helped me in different scenario's where I needed to know the available number of IP addresses within an Azure Virtual Network and Subnet. I know, I know, it doesn't sound very exciting at first, but bear with me - I promise you won't regret it!

So, picture this: you're a cloud engineer, working hard to manage your organization's Azure Virtual Network and Subnets. You've got a lot on your plate, from configuring network security groups to setting up VPN gateways, but there's one task that always seems to elude you: figuring out how many available IP addresses you have in your virtual network and subnets. It's like trying to count the number of grains of sand on a beach - impossible!

Well, fear not my friends, because Kusto Azure Resource Graph queries are here to save the day. With just a few ~~simple~~ commands, you can ~~easily~~ retrieve the number of available IP addresses in your virtual network and subnets, giving you a clear picture of your network's capacity.

First, let's start with the basics. Kusto is a query language used in Azure Data Explorer and Azure Monitor, among other Azure services. It allows you to perform complex queries on your data, using a syntax similar to SQL. Azure Resource Graph, on the other hand, is a service that allows you to explore and query your Azure resources across multiple subscriptions and resource groups.

Now, back to IP addresses. In Azure, every virtual network and subnet has a range of IP addresses associated with it. These ranges can be divided into smaller subnets, each with their own range of IP addresses. When you deploy a virtual machine or other Azure resource into a subnet, it is assigned an IP address from the available pool.

When you view your Azure Virtual Network and Subnet(s) in the Azure Portal you can see how many IP addresses are still available in that Subnet.

![Screenshot of Azure Portal showing the number of available IP addresses for a Virtual Network Subnet](/assets/03-29-2023-02.png)

Wouldn't it be helpful if you could have this information available for all of your Azure Virtual Networks?

Before I show how you can get this information using an Azure Resource Graph Kusto query, let's first start with some basic information on how to calculate the number of IP addresses.

# How to calculate the number of IP addresses?

For example, if we have the IP address 10.1.0.0 with a subnet mask of 255.255.255.0, we can represent this using CIDR (Classless Inter-Domain Routing) notation as 10.1.0.0/24. The "/24" indicates that the first 24 bits are used for the network portion of the address.

To calculate the number of IP addresses in this subnet, we can use the following formula:

Number of IP addresses = 2^(number of bits in the host portion) - 2

This means we need to calculate 2 raised to the power of 8, which is equivalent to 2 multiplied by itself 8 times:

2^8 = 2 x 2 x 2 x 2 x 2 x 2 x 2 x 2

2\^8 = 256

In this case, the number of bits in the host portion is 8 (since there are 32 bits in total and the first 24 bits are used for the network portion), so we can calculate the number of IP addresses as follows:

Number of IP addresses = 2^8 - 2
Number of IP addresses = 256 - 2
Number of IP addresses = 254

Therefore, in this subnet (10.1.0.0/24), there are 254 usable IP addresses. The "-2" in the formula is because the first and last IP addresses in the subnet are reserved for the network address and the broadcast address, respectively, and cannot be assigned to hosts.

In Azure we need to distract 3 more addresses, so in total 5 addresses cannot be used from the total of 256 IP addresses. 

# Azure Resource Graph 

So, how do we use Kusto Azure Resource Graph queries to retrieve the number of available IP addresses in a virtual network and subnet? Let's take a look.

First, we'll need to open up Azure Resource Graph Explorer and run a query against the Azure Resource Graph. The query will look something like this:

```sql
// Return all Azure Virtual Networks
resources
| join kind=leftouter(
    ResourceContainers 
    | where type =~ 'microsoft.resources/subscriptions' 
    | project subscriptionName=name, subscriptionId
) on subscriptionId
| where type =~ 'microsoft.network/virtualNetworks'
```
Let's break down what's happening here. We're querying the Azure Resource Graph for the deployed Azure virtual networks.

Now we can add some more information, like subnet names, subnet prefixes, prefix length, start ip address, end ip address, etc.

```sql
resources
| join kind=leftouter(
    resourcecontainers 
    | where type =~ 'microsoft.resources/subscriptions' 
    | project subscriptionName=name, subscriptionId
) on subscriptionId
| where type =~ 'microsoft.network/virtualNetworks'
| extend addressPrefixes=array_length(properties.addressSpace.addressPrefixes)
| extend vNetAddressSpace=properties.addressSpace.addressPrefixes
| mv-expand subnet=properties.subnets
| extend virtualNetwork = name
| extend subnetPrefix = subnet.properties.addressPrefix
| extend subnets = properties.subnets
| extend subnetName = tostring(subnet.name)
| extend prefixLength = toint(split(subnetPrefix, "/")[1])
| extend addressPrefix = split(subnetPrefix, "/")[0]
| extend numberOfIpAddresses = trim_end(".0",tostring(pow(2, 32 - prefixLength) - 5))
| extend startIp = strcat(strcat_array((array_slice(split(addressPrefix, '.'), 0, 2)),"."), ".", tostring(0))
| extend endIp = strcat(strcat_array((array_slice(split(addressPrefix, '.'), 0, 2)),"."), ".", trim_end(".0",tostring(pow(2, 32 - prefixLength) - 5))) 
| project subscriptionName, resourceGroup, location, virtualNetwork, subnetPrefix, subnetName, prefixLength, numberOfIpAddresses, startIp, endIp
```

When I run this Kusto query in the Azure Resource Graph I get the following results returned.

![Azure Resource Graph results in the Azure Portal](/assets/03-29-2023-01.png)

Let's break down what's happening here. We're querying the Azure Resource Graph for virtual network subnets and combine the resources table with the resourcecontainers table to include the Subscription name to records.

By using the extend operator we create new columns for the properties we want to retrieve. If we look at the prefixLength property this line creates a new column called prefixLength and assigns it the numeric prefix length of each subnet, which is extracted from the subnetPrefix column by splitting it on the '/' character and selecting the second element. The subnetPrefix value "10.1.0.1/24" will result in the prefixLenght value "24"

The most interesting part of the Kusto query is probably where the numberOfIpAddresses column is created. 

```sql
| extend numberOfIpAddresses = trim_end(".0",tostring(pow(2, 32 - prefixLength) - 5))
```

This line creates a new column called numberOfIpAddresses and calculates the number of available IP addresses in each subnet. This is done by subtracting 5 from 2 to the power of the number of bits in the subnet mask (which is 32 minus the prefix length), and then converting the result to a string and trimming the trailing .0 characters. This formula excludes the network address, the broadcast address, and 3 reserved addresses that cannot be assigned to hosts.

Some more columns are added to the final query result and with the last project operator the final columns are presented.

Overall, this query uses a series of join and extend operations to extract and transform information about virtual networks and their subnets in Azure Resource Graph, and then calculates the number of available IP addresses in each subnet based on the subnet prefix length. The resulting table provides a useful summary of the IP address usage in an Azure environment.

And there you have it! With just a few ~~simple~~ commands, you can retrieve the number of available IP addresses in your Azure Virtual Network and Subnets. No more counting grains of sand on the beach.

I hope you learned something new and useful.
