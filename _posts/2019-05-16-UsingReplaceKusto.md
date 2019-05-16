---
layout: post
title: Using Replace Function in Kusto Query Language
categories: [Azure, Kusto, Log Analytics]
tags: [Azure, Kusto, Log Analytics]
comments: true
---

I wanted to replace some string values in one of my Log Analytics Kusto queries and had some difficulty to get the result I was looking for.

In this blog post I'll demonstrate how I got the wanted results.

The <a href="https://docs.microsoft.com/en-us/azure/kusto/query/replacefunction" target="_blank">Kusto Query language has an replace function</a> which replaces all regex matches with another string.

```sql
// Example on replacing strings
datatable(Age:string,FirstName:string,LastName:string)
[
"50","Stefan","Stranger",
"40","John", "Doe",
"30","Jane", "Doe",
]
| extend NewAge=replace(@'50', @'45', Age)
| extend NewAge=replace(@'40', @'35', NewAge)
| extend NewAge=replace(@'30', @'25', NewAge)
```

With above Kusto Query you can replace string values from your data set.

![Kusto Query](/assets/2019-05-16.png)


Hope this was helpfull.

