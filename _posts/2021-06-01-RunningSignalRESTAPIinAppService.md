---
layout: post
title: Running Signal Messenger REST API in Azure Web App for Containers
categories: [Azure, Signal]
tags: [Azure, Signal]
comments: true
---

Why would you want to run the [Signal Messenger](https://www.signal.org/) as an <a href="https://azure.microsoft.com/en-us/services/app-service/" target="_blank">Azure Web App</a>? 

Because you can ;-) No seriously I was looking for way to integrate Signal's messaging capabilities into solutions like [Azure Logic Apps](https://docs.microsoft.com/en-us/azure/logic-apps/) or [Power Automate](https://flow.microsoft.com).

At one of my customers we are using a WhatsApp group to share news about Azure and I'm in the process of moving away from WhatsApp to Signal and this was a good incentive to figure out if I'm able to integrate Signal into some example Logic App flows. If you are looking for reasons to switch to Signal check this [Microsoft customer story](https://azure.microsoft.com/en-us/solutions/confidential-compute/#customer-stories) where Signal is sharing that it's using Azure confidential computing. There might be more reasons for moving or using Signal as your prefered messenger client ;-)

# Scenario

I would like to figure out if the following scenario can be implemented using Signal and Azure Logic Apps.

![Process Flow](/assets/SignalAzureNewProcessFlow.png)

So each time a new Azure news item is published this should be sent to a Signal Messenger group.

## Architecture

![Architecture](/assets/SignalCLIRESTAPIArchitecture.png)

An Azure Logic App is calling the Signal CLI REST API running in an Azure Web App for Containers to send messages the a Signal Messenger Group.


##  Implementation steps

The following high-level steps are used to create this solution.

1. Create Resource Group
1. Create a Web App for Containers
1. Test Signal-CLI-REST-API Web App
1. Create a Logic App
1. Create a Logic App Workflow
1. Add Authentication to Web App
1. Update Logic App workflow with Authentication information
1. Create end-to-end Logic App Workflow

### Create Resource Group

Create the Azure Resource Group where you want to deploy your Azure Web App and Logic App. See [here](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal#create-resource-groups) for examples on creating resource groups.

### Deploy and configure the Azure Web (for Containers)

With Web App for Containers you can easily deploy and run containerised applications on Windows and Linux. In our case we are going to run the <a href="https://github.com/bbernh
ard/signal-cli-rest-api" target="_blank">Dockerized Signal Messenger REST API</a> in an Azure Web App for Containers.

Go to the Azure Portal and select Web App for Containers in the Azure Marketplace and click on Create.

![](/assets/06-11-2021-01.png) 

You now need to configure the Container settings of the Web App. Enter below information in these setting and save the setting. Use for the Docker Container image: bbernhard/signal-cli-rest-api:latest

![Web App Configuration](/assets/2021-06-10_17-15-36.png)

And click on Review + Create.

Wait till the deployment of the Web App is finished and continue with the configuration.

### Link Signal CLI REST API Web App 

To test the Signal-CLI-REST-API Web App we first need to link the Web App as a Signal Device.

To do so you need to call the following endpoint:

https://[name of your webapp].azurewebsites.net/v1/qrcodelink?device_name=[name of linked device]

E.g.
https://demo-for-signal-cli-rest-api.azurewebsites.net/v1/qrcodelink?device_name=webapp

Open the url in your browser. 

![Web App Configuration](/assets/2021-06-10_17-29-47.png)

And scan the qrcode with your mobile Signal App via the menu.

<a href="/assets/2021-06-10_17-32-31.png"><img width="540" height="1170" title="image" style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border-width: 0px" alt="image" src="..//assets/2021-06-10_17-32-31.png" border="0" /></a>

After a succesful link of the Device you should see the something like this in your Signal Mobile App.

<a href="/assets/14-06-2021-01.png"><img width="540" height="1170" title="image" style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border-width: 0px" alt="image" src="..//assets/14-06-2021-01.png" border="0" /></a>

### Test Signal-CLI-REST-API

You can now start testing receiving and sending Signal Messages using the REST API.

Sample Request

| Receive |
|----------|
| HTTPS |
| GET https://[name of web app].azurewebsites.net/v1/receive/[mobile phone number plus country code. E.g. +316xxxxxxxx]  |

Request Body

<table>
  [Empty]
</table>

Content Type of the request: Not Applicable

Sample Response

Status code: 200

<table>
<td>
<p>[{"envelope":{"source":"+316xxxxxxxx","sourceDevice":1,"timestamp":1623674475537,"syncMessage":{"type":"GROUPS_SYNC"}}},{"envelope":{"source":"+316xxxxxxxx","sourceDevice":1,"timestamp":1623674475672,"syncMessage":{"blockedNumbers":[],"blockedGroupIds":[]}}},{"envelope":{"source":"+316xxxxxxxx","sourceDevice":1,"timestamp":1623674476281,"syncMessage":{}}},{"envelope":{"source":"+316xxxxxxxx","sourceDevice":1,"timestamp":1623674476328,"syncMessage":{}}},{"envelope":{"source":"+316xxxxxxxx","sourceDevice":1,"timestamp":1623674476909,"syncMessage":{}}},{"envelope":{"source":"+316xxxxxxxx3","sourceDevice":1,"timestamp":1623674477024,"syncMessage":{"type":"CONTACTS_SYNC"}}}]</p>
</td>
</table>

Status code: 400
<table>
<td>
User +316xxxxxxxx is not registered.
</td>
</table>

Solution for " 316xxxxxxxx is not registered"  is unlinking Web App Device and relinking Device again.

Sample Request

| Send Message |
|----------|
| HTTPS |
| POST https://[name of web app].azurewebsites.net/v2/send |

Request Body

<table>
  <td width="601" valign="top" style="width: 450.8pt; padding: 5px;">
  <p>{<br />"message": "Hello World from Signal-CLI-REST-API Azure Web App",<br />"number": "+316xxxxxxxx",<br />"recipients": [<br />"+316xxxxxxxx"<br />]<br />}</p>
  </td>
</table>

Content Type of the request: application/json

Sample Response

Status code: 200

<table>
<td>
<p>{<br />"timestamp": "1623675022528"<br />}</p>
</td>
</table>

Status code: 400
<table>
<td>
User +316xxxxxxxx is not registered.
</td>
</table>

Solution for " 316xxxxxxxx is not registered"  is unlinking Web App Device and relinking Device again.

You can use tools like [HttpMaster](https://httpmaster.net/) to test your Signal-CLI-REST-API Web App.

![HttpMaster send message call](/assets/2021-06-13_15-29-28.png)

On you Signal Mobile App you should see the message appear.

<a href="/assets/14-06-2021-02.png"><img width="540" height="1170" title="image" style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border-width: 0px" alt="image" src="/assets/14-06-2021-02.png" border="0" /></a>

### Logic App installation and configuration

Follow the steps describe [here](https://docs.microsoft.com/en-us/azure/logic-apps/quickstart-create-first-logic-app-workflow) to create a logic app using the Azure Portal.

After you have created the Azure Logic app go to the resource and open the Logic App Designer.

The end goal is to create the following Logic App Workflow.

![Logic App Workflow](/assets/2021-06-14_15-12-16.png)

Start with selecting a Blank Logic App and search for the Schedule trigger and select Recurrence Schedule trigger.

![Schedule trigger](/assets/2021-06-14_16-12-48.png)

![Configure Schedule trigger](/assets/14-06-2021-03.png)

Next add the HTTP Built-in Action.

![HTTP request Action](/assets/2021-06-14_16-15-48.png)

Configure the following settings in the HTTP request trigger.

![Configure HTTP request Action](/assets/2021-06-14_16-18-42.png)

Add the List all RSS feed items action as a next step and configure as shown below. I've used the following RSS feed url: 'https://techcommunity.microsoft.com/gxcuf89792/rss/Community?interaction.style=blog'

![Add and configure List all RSS feed Items Action](/assets/2021-06-14_16-24-05.png) 

The final steps are executed in the for each Control Action

![Add and configure for each control action](/assets/2021-06-14_16-35-14.png) 

![Add and configure for each control action](/assets/2021-06-14_16-36-54.png) 

You need to add an extra delay to make sure the Signal-CLI-REST-API does not get too many requests at the same time.

But we are not completely finished. As a last step we are adding Authentication on the Web App to make sure only authenticated users can call the REST API running in the Web App.

### Authentication for the Web App

To enable Authentication on the Web App you need to go to the Authentication setting.

![Enable Authentication on Web App](/assets/2021-06-13_14-50-32.png) 

Click on Add Identity provider. Select Microsoft as Identity Provider

![Configure Identity Provider](/assets/2021-06-13_14-51-44.png) 

Skip any permissions for now.

![Identity Provider permissions](/assets/14-06-2021-04.png)

Copy AppId/ClientID

![AppID](/assets/2021-06-13_14-53-52.png)

Change the **accessTokenAcceptedVersion** setting in the App Registration's manifest file from **null to 2**. If you don't do this you might end up with the following error message when calling the REST API:

"IDX10205: Issuer validation failed. Issuer: '[PII is hidden]'. Did not match: validationParameters.ValidIssuer: '[PII is hidden]' or validationParameters.ValidIssuers: '[PII is hidden]'.""

![App Registration Manifest file](/assets/2021-06-14_16-53-23.png)

The final step after enabling the Authentication on the Web App is updating the Logic App Workflow actions to include the authentication.

First turn on the System Assigned identity in the Logic App.

![Logic App Managed Identity](/assets/2021-06-14_16-57-30.png)

The final steps are updating all HTTP Request actions with the authorization configuration. Select Managed Identity as Authentication type and use the earlier copied AppID (ClientID) as Audience setting.

![Authentication setting for HTTP Request action](/assets/2021-06-14_16-59-30.png)

Repeat this for all the HTTP Requests in your Logic App Workflow!

If you want to test the authentication on your Web App using a REST API client like HttpMaster you first need to get an Access Token and use that Access Token as Authentication Bearer Header in the REST API call.

Here is an example:

You need to have a SPN created in your tenant and have the following properties of the SPN:
* ClientID
* ClientSecret
* TenantId

Sample Request

| Get Access Token|
|----------|
| HTTPS |
| POST https://login.windows.net/{TenantID}/oauth2/token |

Request Body

<table>
  resource={AppID}&client_id={ClientID}&grant_type=client_credentials&client_secret={ClientSecret}
</table>

Content Type of the request: application/x-www-form-urlencoded

Sample Response

Status code: 200

<table>
<td>
<p>{<br />"token_type": "Bearer",<br />"expires_in": "3599",<br />"ext_expires_in": "3599",<br />"expires_on": "1623686877",<br />"not_before": "1623682977",<br />"resource": "a49f4c94-475e-4f18-a643-970b90bea30e",<br />"access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Im5PbzNaRHJPRFhFSzFqS1doWHNsSFJfS1hFZyJ9.eyJhdWQiOiJhNDlmNGM5NC00NzVlLTRmMTgtYTY0My05NzBiOTBiZWEzMGUiLCJpc3MiOiJodHRwczovL2xvZ2luLm1pY3Jvc29mdG9ubGluZS5jb20vNDk2ZjBiMjctNGZhNC00YzNkLThiYmUtMTljNGI2ODc1YzgxL3YyLjAiLCJpYXQiOjMzY4Mjk3NywiZXhwIjoxNjIzNjg2ODc3LCJhaW8iOiJFMlpnWUJCc1hpbXV1YjJncW5yR29qOVRxdVN2QVFBPSIsImF6cCI6IjRlYTJiYmQ5LWJkMTMtNDY1Ni1hYWJhLWViMTU4ZDMzNTgwNyIsImF6cGFjciI6IjEiLCJvaWQiOiJlNzhjMmQ2Yi1iODcwLTQwYmUtOGI0NS03M2EyMTQ1NGFlYjIiLCJyaCI6IjAuQVFzQUp3dHZTYVJQUFV5THZobkV0b2RjZ2RtN29rNFR2VlpHcXJyckZZMHpXQWNMQUFBLiIsInN1YiI6ImU3OGMyZDZiLWI4NzAtNDBiZS04YjQ1LTczYTIxNDU0YWViMiIsInRpZCI6IjQ5NmYwYjI3LTRmYTQtlLTE5YzRiNjg3NWM4MSIsInV0aSI6ImRqTVViWHNuQVVlTk53V2xGSGN2QUEiLCJ2ZXIiOiIyLjAifQ.VFaWOUwfHBzdH83KMlSIXWR6P4AZkMBz5Hgtp3MrIaQez_uVgCdGMYBg63kXuvreSABc5-fDvXqGeVdlB6P5RF3eTIDXD-fGn7yfBSRmliqp88zMN4PUpqJpPnmQ0k6fzQwJ-OGz3pwGiLouEtFedLVPgQRBMD6gY1wl3I_LsIsaJcJhdoy-sJt4YdVDBbZ9ubHQ42-g3RMkQxRnZ8dzzVlHt9UWaPdugALxFCGa4_ADBMrq2u4ix0lKVn6JWkSIpEi-xkf7vMLL9P00l0uV6STKP_QEr8tm2aduES6N7MwbmTSFYO6P7xFiig73CPq5pYdVVhIE0lj4lu2M2Z8Vxw"<br />}</p>
</td>
</table>

You can use this Access Token in the call to the REST API as follows:

| Send Message |
|----------|
| HTTPS |
| POST https://[name of web app].azurewebsites.net/v2/send |

Headers

<table>
  <td width="601" valign="top" style="width: 450.8pt; padding: 5px;">
  <p>Authorization: bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Im5PbzNaRHJPRFhFSzFqS1doWHNsSFJfS1hFZyJ9.eyJhdWQiOiJhNDlmNGM5NC00NzVlLTRmMTgtYTY0My05NzBiOTBiZWEzMGUiLCJpc3MiOiJodHRwczovL2xvZ2luLm1pY3Jvc29mdG9ubGluZS5jb20vNDk2ZjBiMjctNGZhNC00YzNkLThiYmUtMTljNGI2ODc1YzgxL3YyLjAiLCJpYXQiOjMzY4Mjk3NywiZXhwIjoxNjIzNjg2ODc3LCJhaW8iOiJFMlpnWUJCc1hpbXV1YjJncW5yR29qOVRxdVN2QVFBPSIsImF6cCI6IjRlYTJiYmQ5LWJkMTMtNDY1Ni1hYWJhLWViMTU4ZDMzNTgwNyIsImF6cGFjciI6IjEiLCJvaWQiOiJlNzhjMmQ2Yi1iODcwLTQwYmUtOGI0NS03M2EyMTQ1NGFlYjIiLCJyaCI6IjAuQVFzQUp3dHZTYVJQUFV5THZobkV0b2RjZ2RtN29rNFR2VlpHcXJyckZZMHpXQWNMQUFBLiIsInN1YiI6ImU3OGMyZDZiLWI4NzAtNDBiZS04YjQ1LTczYTIxNDU0YWViMiIsInRpZCI6IjQ5NmYwYjI3LTRmYTQtlLTE5YzRiNjg3NWM4MSIsInV0aSI6ImRqTVViWHNuQVVlTk53V2xGSGN2QUEiLCJ2ZXIiOiIyLjAifQ.VFaWOUwfHBzdH83KMlSIXWR6P4AZkMBz5Hgtp3MrIaQez_uVgCdGMYBg63kXuvreSABc5-fDvXqGeVdlB6P5RF3eTIDXD-fGn7yfBSRmliqp88zMN4PUpqJpPnmQ0k6fzQwJ-OGz3pwGiLouEtFedLVPgQRBMD6gY1wl3I_LsIsaJcJhdoy-sJt4YdVDBbZ9ubHQ42-g3RMkQxRnZ8dzzVlHt9UWaPdugALxFCGa4_ADBMrq2u4ix0lKVn6JWkSIpEi-xkf7vMLL9P00l0uV6STKP_QEr8tm2aduES6N7MwbmTSFYO6P7xFiig73CPq5pYdVVhIE0lj4lu2M2Z8Vxw</p>
  </td>
</table>


Request Body

<table>
  <td width="601" valign="top" style="width: 450.8pt; padding: 5px;">
  <p>{<br />"message": "Hello World from Signal-CLI-REST-API Azure Web App",<br />"number": "+316xxxxxxxx",<br />"recipients": [<br />"+316xxxxxxxx"<br />]<br />}</p>
  </td>
</table>

Content Type of the request: application/json

Sample Response

Status code: 200

<table>
<td>
<p>{<br />"timestamp": "1623675022528"<br />}</p>
</td>
</table>

Status code: 400
<table>
<td>
User +316xxxxxxxx is not registered.
</td>
</table>

Hope you learned something new.

# References:

- [Signal website](https://www.signal.org/)
- [Dockerized Signal Messenger REST API](https://github.com/bbernhard/signal-cli-rest-api)
- [Swagger Signal CLI REST API documentation](https://bbernhard.github.io/signal-cli-rest-api/)
- [Azure Logic Apps](https://docs.microsoft.com/en-us/azure/logic-apps/)
- [Power Automate](https://flow.microsoft.com)
- [Microsoft identity platform access tokens](https://docs.microsoft.com/en-us/azure/active-directory/develop/access-tokens)
