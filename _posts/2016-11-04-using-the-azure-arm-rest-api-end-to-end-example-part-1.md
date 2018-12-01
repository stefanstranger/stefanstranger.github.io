---
layout: post
title: Using the Azure ARM REST API – End to end Example Part 1
date: 2016-11-04 14:05
author: stefan stranger
comments: true
categories: [ARM, Automation, Azure, Azure, REST API]
---
In the first two blog post about using the Azure (ARM) REST API I explained how to get the <a target="_blank" href="https://blogs.technet.microsoft.com/stefan_stranger/2016/10/21/using-the-azure-arm-rest-apin-get-access-token/">Access Token</a> and how to get some <a target="_blank" href="https://blogs.technet.microsoft.com/stefan_stranger/2016/10/29/using-the-azure-arm-rest-api/">simple info about your Azure Subscription</a>.

In the last two blog posts about using the Azure (ARM) REST API we are going to do the following:
<ol>
 	<li>Create a Resource Group</li>
 	<li>Create a Virtual Machine using an ARM template</li>
 	<li>Call an Azure Automation Runbook to stop the Virtual Machine</li>
</ol>
And all these steps are being executed by plain web requests against the Azure (ARM) REST APIs.

<strong>Pre-requisites:</strong>
<ul>
 	<li>Azure Automation already setup</li>
 	<li>Azure Runbook for stopping the VM already created.</li>
</ul>
<strong>Tooling:</strong>

There are a number of tools you can use for making web requests, like:
<ul>
 	<li><a target="_blank" href="https://www.getpostman.com/">Postman</a></li>
 	<li><a target="_blank" href="http://httpmaster.net/">HTTPMaster</a></li>
</ul>
I like to work with HTTPMaster and that’s why I will use it in this blog post.

HttpMaster has the following great features:
<ul>
 	<li>Web API Tool for development and testing of API applications &amp; services</li>
 	<li>Web Service Tool with complete support for testing RESTful web services</li>
 	<li>Website Testing Tool focused on posting HTML forms and uploading files</li>
 	<li>Universal Http Tool to simulate client activity for any web application type</li>
</ul>
Just download and install the <a target="_blank" href="http://httpmaster.net/download">Free Express Edition</a> to get started.
<ol>
 	<li><strong>Create the Resource Group</strong></li>
</ol>
We will look at the <a target="_blank" href="https://msdn.microsoft.com/en-us/library/azure/dn790525.aspx">Azure Reference</a> web site for more information on creating a Resource Group.

The request needs to look like this:
<table width="669" border="0" cellspacing="0" cellpadding="2">
<tbody>
<tr>
<td width="155" valign="top"><strong>Method</strong></td>
<td width="512" valign="top"><strong>Request URI</strong></td>
</tr>
<tr>
<td width="155" valign="top">PUT</td>
<td width="512" valign="top"><a href="https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}?api-version={api-version">https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}?api-version={api-version</a>}</td>
</tr>
</tbody>
</table>
&nbsp;

Request Body:
<table width="400" border="0" cellspacing="0" cellpadding="2">
<tbody>
<tr>
<td width="400" valign="top"><strong>JSON</strong></td>
</tr>
<tr>
<td width="400" valign="top">{
"location": "West US",
"tags": {
"tagname1": "tagvalue1"
}
}</td>
</tr>
</tbody>
</table>
&nbsp;

Open HttpMaster and create a new Project.

<a href="https://msdnshared.blob.core.windows.net/media/2016/11/image153.png"><img width="811" height="329" title="image" style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb139.png" border="0" /></a>

Enter project properties: (use as global URL: <a href="https://management.azure.com/subscriptions" title="https://management.azure.com/subscriptions">https://management.azure.com/subscriptions</a>) and click on OK.

<a href="https://msdnshared.blob.core.windows.net/media/2016/11/image154.png"><img width="885" height="568" title="image" style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb140.png" border="0" /></a>

&nbsp;

Add a new item to your Project.

<a href="https://msdnshared.blob.core.windows.net/media/2016/11/image155.png"><img width="746" height="370" title="image" style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb141.png" border="0" /></a>

We first need to retrieve the SAS Token which I explained in my <a target="_blank" href="https://blogs.technet.microsoft.com/stefan_stranger/2016/10/21/using-the-azure-arm-rest-apin-get-access-token/">first blog post</a>.

<a href="https://msdnshared.blob.core.windows.net/media/2016/11/image156.png"><img width="885" height="446" title="image" style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb142.png" border="0" /></a>

All the values between the curly brackets are parameters you can use in your HttpMaster project.

Create the following parameters in your project:
<ul>
 	<li>TenantId (enter your value in the value field of HttpMaster)</li>
 	<li>ARMResource (<a href="https://management.core.windows.net/" title="https://management.core.windows.net/">https://management.core.windows.net/</a>)</li>
 	<li>ClientID (enter your value in the value field of HttpMaster)</li>
 	<li>ClientSecret (enter your value in the value field of HttpMaster)</li>
 	<li>SubscriptionID (enter your value in the value field of HttpMaster)</li>
</ul>
&nbsp;

<a href="https://msdnshared.blob.core.windows.net/media/2016/11/image157.png"><img width="885" height="564" title="image" style="padding-top: 0px;padding-left: 0px;padding-right: 0px;border: 0px" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb143.png" border="0" /></a>

Enter your Azure Subscription values for these parameters in your HTTPMaster project.

Save your project and execute the Get Access Token request.

<a href="https://msdnshared.blob.core.windows.net/media/2016/11/image158.png"><img width="885" height="461" title="image" style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb144.png" border="0" /></a>

And now you have you access token returned.

<a href="https://msdnshared.blob.core.windows.net/media/2016/11/image159.png"><img width="800" height="572" title="image" style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb145.png" border="0" /></a>

We need this access token in our next request so copy the value of the access token to <a target="_blank" href="https://notepad-plus-plus.org/">Notepad++</a> or another editor of your choice  and add <strong>bearer</strong> to the beginning of the string.

<a href="https://msdnshared.blob.core.windows.net/media/2016/11/image160.png"><img width="885" height="88" title="image" style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb146.png" border="0" /></a>

Copy the complete string to the Header section of HTTPMaster’s Project properties and create a new header with a Name <strong>Authorization</strong> (keep in mind the token is valid for an hour)

<a href="https://msdnshared.blob.core.windows.net/media/2016/11/image161.png"><img width="885" height="256" title="image" style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb147.png" border="0" /></a>

Now we are going to create a new request for <strong>creating the Resource Group</strong>.

You need to following information:
<ul>
 	<li>URL: {SubscriptionId}/resourceGroups/{NewResourceGroupName}?api-version=2014-04-01</li>
 	<li>Body:
<table width="400" border="0" cellspacing="0" cellpadding="2">
<tbody>
<tr>
<td width="400" valign="top">{
"properties": {},
"location": "West Europe"
}</td>
</tr>
</tbody>
</table>
</li>
 	<li>New parameter NewResourceGroupName. (this is new resource group we are going to create)</li>
</ul>
<a href="https://msdnshared.blob.core.windows.net/media/2016/11/image162.png"><img width="633" height="270" title="image" style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb148.png" border="0" /></a>

<a href="https://msdnshared.blob.core.windows.net/media/2016/11/image163.png"><img width="736" height="572" title="image" style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb149.png" border="0" /></a>

Now execute the Create Resource Group request.

<a href="https://msdnshared.blob.core.windows.net/media/2016/11/image164.png"><img width="885" height="390" title="image" style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb150.png" border="0" /></a>

When the result is returned we see that the Resource Group is created.

<a href="https://msdnshared.blob.core.windows.net/media/2016/11/image165.png"><img width="803" height="572" title="image" style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb151.png" border="0" /></a>

Let’s also check in the Azure Portal.

<a href="https://msdnshared.blob.core.windows.net/media/2016/11/image166.png"><img width="885" height="297" title="image" style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" alt="image" src="https://msdnshared.blob.core.windows.net/media/2016/11/image_thumb152.png" border="0" /></a>

Success <img class="wlEmoticon wlEmoticon-smile" style="border-style: none" alt="Smile" src="https://msdnshared.blob.core.windows.net/media/2016/11/wlEmoticon-smile1.png" />

You can download the <a target="_blank" href="https://github.com/stefanstranger/AzureARMRESTAPI/blob/master/HttpMaster/Azure%20REST%20API%20Demo.hmpr">HttpMaster project from my Github Account</a> and use that as a starting point.

Watch my blog for the next blog post about this topic.

<strong>References:</strong>
<ul>
 	<li><a target="_blank" href="http://httpmaster.net/">HttpMaster</a></li>
 	<li><a target="_blank" href="https://msdn.microsoft.com/en-us/library/azure/dn790525.aspx">Azure Reference</a></li>
 	<li><a target="_blank" href="https://blogs.technet.microsoft.com/stefan_stranger/2016/10/21/using-the-azure-arm-rest-apin-get-access-token/">Using the Azure ARM REST API – Get Access Token</a> (Part 1)</li>
 	<li><a target="_blank" href="https://blogs.technet.microsoft.com/stefan_stranger/2016/10/29/using-the-azure-arm-rest-api/">Using the Azure ARM REST API – Get Subscription Information</a> (Part 2)</li>
 	<li><a target="_blank" href="https://github.com/stefanstranger/AzureARMRESTAPI/blob/master/HttpMaster/Azure%20REST%20API%20Demo.hmpr">HttpMaster project from my Github Account</a>.</li>
</ul>
