---
layout: post
title: Using the Azure ARM REST API – Get Subscription Information
date: 2016-10-29 06:05
author: stefan stranger
comments: true
categories: [ARM, Azure, Azure, Linux, open source, PowerShell, PowerShell, Scripts]
---
In the <a target="_blank" href="https://blogs.technet.microsoft.com/stefan_stranger/2016/10/21/using-the-azure-arm-rest-apin-get-access-token/">fist blog post</a> over using the Azure ARM REST API I explained how to retrieve the Access Token needed for the further authentication against the Azure ARM REST API.

In this blog post I’m going to explain how you can use that Access Token and start communicating with Azure using simple web calls.

<strong>How to use the Access Token for Authentication?</strong>

It took some time to find the correct location about how to use the Azure REST APIs but a good starting point is the <a target="_blank" href="https://msdn.microsoft.com/en-us/library/azure/mt420159.aspx">Azure Reference on MSDN</a>.

For the authentication part I found information on a blog post from <a target="_blank" href="https://twitter.com/davidebbo">David Ebbo</a> called “<a target="_blank" href="http://blog.davidebbo.com/2015/12/calling-arm-using-plain-rest.html">Calling the Azure ARM API using plain REST</a>”

The interesting part for us is how the Request Header should look like.

<a href="https://stefanstranger.github.io/assets/image1076.png"><img width="754" height="263" title="image" style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border-width: 0px" alt="image" src="https://stefanstranger.github.io/assets/image_thumb930.png" border="0" /></a>

So this shows us that the when we have the Access Token we need to create a web Request Header with the following info:
<table width="401" border="1" cellspacing="0" cellpadding="2">
<tbody>
<tr>
<td width="199" valign="top"><strong>Authorization</strong></td>
<td width="200" valign="top"><strong>Bearer “[AccessToken]”</strong></td>
</tr>
</tbody>
</table>
&nbsp;

<strong>Get information about a subscription</strong>

Ok now we know how to use the Access Token we can start with a simple get info about the Azure Subscription. And again you can find info on retrieving that info on the <a target="_blank" href="https://msdn.microsoft.com/en-us/library/azure/dn790579.aspx">Azure Reference links</a>.

For retrieving the <a target="_blank" href="https://msdn.microsoft.com/en-us/library/azure/dn790579.aspx">Subscription information</a> we need to use the following request URI.

&nbsp;

<a href="https://stefanstranger.github.io/assets/image1077.png"><img width="789" height="359" title="image" style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border-width: 0px" alt="image" src="https://stefanstranger.github.io/assets/image_thumb931.png" border="0" /></a>

Let’s do a web request call using PowerShell Invoke-RestMethod cmdlet first.
<pre class="”brush:">#requires -Version 3

# ---------------------------------------------------
# Script: C:\Scripts\GetAzureSubscriptionRESTAPI.ps1
# Version:
# Author: Stefan Stranger
# Date: 10/28/2016 15:16:25
# Description: Get Azure Subscription Info using plain REST API calls.
# Comments:
# Changes:  
# Disclaimer: 
# This example is provided "AS IS" with no warranty expressed or implied. Run at your own risk. 
# **Always test in your lab first**  Do this at your own risk!! 
# The author will not be held responsible for any damage you incur when making these changes!
# ---------------------------------------------------


#region variables SPN ClientId and Secret
$ClientID       = '[ClientID]' #ApplicationID
$ClientSecret   = '[ClientSecret]'  #key from Application
$tennantid      = '[TennantID]'
$SubscriptionId = '[Subscription]'
#endregion
 

#region Get Access Token
$TokenEndpoint = {https://login.windows.net/{0}/oauth2/token} -f $tennantid 
$ARMResource = "https://management.core.windows.net/";

$Body = @{
        'resource'= $ARMResource
        'client_id' = $ClientID
        'grant_type' = 'client_credentials'
        'client_secret' = $ClientSecret
}

$params = @{
    ContentType = 'application/x-www-form-urlencoded'
    Headers = @{'accept'='application/json'}
    Body = $Body
    Method = 'Post'
    URI = $TokenEndpoint
}

$token = Invoke-RestMethod @params
#endregion

#region Get Azure Subscription
$SubscriptionURI = "https://management.azure.com/subscriptions/$SubscriptionID" +'?api-version=2016-09-01'

$params = @{
    ContentType = 'application/x-www-form-urlencoded'
    Headers = @{
    'authorization'="Bearer $($Token.access_token)"
    }
    Method = 'Get'
    URI = $SubscriptionURI
}

Invoke-RestMethod @params
#endregion
</pre>
When running above PowerShell script we receive the following info about the Azure Subscription.

<a href="https://stefanstranger.github.io/assets/image1078.png"><img width="885" height="147" title="image" style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border-width: 0px" alt="image" src="https://stefanstranger.github.io/assets/image_thumb932.png" border="0" /></a>

Because we are using plain REST API web calls we can use all kind of tools, like for instance Bash scripts.

As you know <a target="_blank" href="https://blogs.technet.microsoft.com/windowsserver/2015/05/06/microsoft-loves-linux/">Microsoft Loves Linux</a> and we can use <a target="_blank" href="https://blogs.windows.com/buildingapps/2016/03/30/run-bash-on-ubuntu-on-windows/">Bash on Windows</a> (if you are on the Windows 10 Insider builds) to create a Bash script and use Curl to retrieve the Azure Subscription information.

<strong>Remarks</strong>:
<ul>
 	<li>I installed <a target="_blank" href="https://stedolan.github.io/jq/">jq, a lightweight and flexible command-line JSON processor</a> to parse the JSON output from curl, using apt-get install jq on Bash on Windows.</li>
 	<li>If you are creating the getazuresubscription.sh Bash script on Bash on Windows you need to make sure you save the file as unix file type.
In VIM you can do that with <code>:set ff=unix</code></li>
</ul>
getazuresubscription.sh file:
<pre class="”brush:">#!/bin/bash

# bash script to retrieve Azure Subscription information using plain Azure ARM REST API web requests

#Azure Subscription variables
ClientID="[ClientID]" #ApplicationID
ClientSecret="[ClientSecret]"  #key from Application
TennantID="[TennantID]"
SubscriptionID="[SubscriptionID]"

accesstoken=$(curl -s --header "accept: application/json" --request POST "https://login.windows.net/$TennantID/oauth2/token" --data-urlencode "resource=https://management.core.windows.net/" --data-urlencode "client_id=$ClientID" --data-urlencode "grant_type=client_credentials" --data-urlencode "client_secret=$ClientSecret" | jq -r '.access_token')

#Use AccessToken in Azure ARM REST API call for Subscription Info
subscriptionURI="https://management.azure.com/subscriptions/$SubscriptionID?api-version=2016-09-01"

curl -s --header "authorization: Bearer $accesstoken" --request GET $subscriptionURI | jq .
</pre>
When you run above script from Bash on Windows you get the following output returned:

<a href="https://stefanstranger.github.io/assets/basharmrestapi.gif"><img width="1789" height="786" title="basharmrestapi" alt="basharmrestapi" src="https://stefanstranger.github.io/assets/basharmrestapi_thumb.gif" /></a>

How cool is that?

In the last example of this blog post we are going to use Javascript to do the same as the previous examples.

I use <a target="_blank" href="https://code.visualstudio.com">Visual Studio Code</a> to develop most of my scripts lately, you can also use that for the PowerShell and Bash scripts creation if you want. It even runs on Linux and a Mac <img class="wlEmoticon wlEmoticon-smile" style="border-style: none" alt="Smile" src="https://stefanstranger.github.io/assets/wlEmoticon-smile14.png" />

&nbsp;

<a href="https://stefanstranger.github.io/assets/image1079.png"><img width="1603" height="862" title="image" style="padding-top: 0px;padding-left: 0px;padding-right: 0px;border-width: 0px" alt="image" src="https://stefanstranger.github.io/assets/image_thumb933.png" border="0" /></a>

<strong></strong>

GetAzureSubscription.js file:
<pre class="”brush:">/*
    Author: Stefan Stranger
    Date: 10/24/2016
    Description: Use Javascript to retrieve Azure Subscription information.
    More info: https://blogs.technet.microsoft.com/stefan_stranger/2016/10/21/using-the-azure-arm-rest-apin-get-access-token/
*/
var request, options;
request = require('request');

//Helper Function
function AzureARMAccessToken(ClientID, ClientSecret, TennantID, callback) {
    options = {
        url: 'https://login.windows.net/' + TennantID + '/oauth2/token', //URL to hit
        method: 'POST',
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'accept': 'application/json'
        },
        body: 'resource=' + encodeURIComponent('https://management.core.windows.net/') + '&amp;client_id=' + ClientID + '&amp;grant_type=client_credentials&amp;client_secret=' + encodeURIComponent(ClientSecret),

    };
    //Start the request
    request(options, function (error, response, body) {
        if (!error &amp;&amp; response.statusCode == 200) {
            callback(body);
        }
        else
            console.log(error);


    });
}


//Function to Get Azure Subscription Information
function GetAzureSubscription(clientID, clientSecret, tennantID, subscriptionID) {
    AzureARMAccessToken(clientID, clientSecret, tennantID, function (data) {
        var jsonData = JSON.parse(data);
        var accessToken = 'bearer ' + jsonData.access_token
        //Get StorageAccount info
        options = {
            url: 'https://management.azure.com/subscriptions/' + subscriptionID + '?api-version=2016-09-01',
            method: 'GET',
            headers: {
                'Authorization': accessToken,
                'accept': 'application/json'
            },
        };
        //Start the request
        request(options, function (error, response, body) {
            if (!error &amp;&amp; response.statusCode == 200) {
                var jsonData = JSON.parse(body);
                console.log(jsonData);
            }
            else
                console.log(error);


        });

    })
}

//Main
myClientID = "[ClientID]";
myClientSecret = "[ClientSecret]";
myTennantID = "[TennnantID]";
mySubscriptionID = "[SubscriptionID]"


GetAzureSubscription(myClientID, myClientSecret, myTennantID, mySubscriptionID);
</pre>
I hope the above examples showed why it is cool to use Azure (ARM) REST APIs to manage Azure. In the next blog post I’m going to explore the Azure (ARM) REST API a little more.

Have fun!

<strong>References:</strong>
<ul>
 	<li>Azure Reference: <a href="https://msdn.microsoft.com/en-us/library/azure/mt420159.aspx" title="https://msdn.microsoft.com/en-us/library/azure/mt420159.aspx">https://msdn.microsoft.com/en-us/library/azure/mt420159.aspx</a></li>
 	<li>API Management REST: <a href="https://msdn.microsoft.com/en-us/library/azure/dn776326.aspx" title="https://msdn.microsoft.com/en-us/library/azure/dn776326.aspx">https://msdn.microsoft.com/en-us/library/azure/dn776326.aspx</a></li>
 	<li>Azure Resource Manager REST API Reference: <a href="https://msdn.microsoft.com/en-us/library/azure/dn790568.aspx" title="https://msdn.microsoft.com/en-us/library/azure/dn790568.aspx">https://msdn.microsoft.com/en-us/library/azure/dn790568.aspx</a></li>
 	<li>Calling the Azure ARM API using plain REST: <a href="http://blog.davidebbo.com/2015/12/calling-arm-using-plain-rest.html" title="http://blog.davidebbo.com/2015/12/calling-arm-using-plain-rest.html">http://blog.davidebbo.com/2015/12/calling-arm-using-plain-rest.html</a></li>
 	<li>Microsoft Loves Linux: <a href="https://blogs.technet.microsoft.com/windowsserver/2015/05/06/microsoft-loves-linux/" title="https://blogs.technet.microsoft.com/windowsserver/2015/05/06/microsoft-loves-linux/">https://blogs.technet.microsoft.com/windowsserver/2015/05/06/microsoft-loves-linux/</a></li>
 	<li>jq, a lightweight and flexible command-line JSON processor: <a href="https://stedolan.github.io/jq/" title="https://stedolan.github.io/jq/">https://stedolan.github.io/jq/</a></li>
 	<li>Visual Studio Code: <a href="https://code.visualstudio.com" title="https://code.visualstudio.com">https://code.visualstudio.com</a></li>
</ul>
