---
layout: post
title: Azure Service Health Issue JIRA Integration
categories: [Azure, JIRA, Logic App]
tags: [Azure, JIRA, Logic App]
comments: true
---


- [Introduction](#introduction)
  - [Azure Service Health Issues](#azure-service-health-issues)
  - [JIRA Service Management](#jira-service-management)
  - [Example JIRA REST API calls used in the Logic App Workflow](#example-jira-rest-api-calls-used-in-the-logic-app-workflow)
    - [Get JIRA Fields](#get-jira-fields)
    - [Get JIRA Issue Types](#get-jira-issue-types)
- [Log Analytics Query](#log-analytics-query)
- [Log Analytics Workflow](#log-analytics-workflow)
- [FAQ](#faq)
- [References](#references)


# Introduction

I've been asked to develop a solution that manages JIRA Incidents in JIRA Service Management for Azure Service Health Issues.

## Azure Service Health Issues

Azure Service Health notifies you about Azure service incidents and planned maintenance so you can take action to mitigate downtime. 

You can configure Alerts for the following Service health events (Service issues, Planned Maintenance, Health advisories and Security advisories).

Here is an example of an Azure Service Health Alert from the Azure Portal.

![Screenshot of Azure Service Health Issue in Azure Portal](/assets/10-05-2022-01.png)

Service health notifications are stored in the [Azure activity log](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/platform-logs-overview). Given the large volume of information stored in the activity log, there is a separate user interface to make it easier to view and set up alerts on service health notifications.

Here is a screenshot of the same Azure Service Health Alert Issue information stored in the Log Analytics Workspace for each of the impacted Azure Subscriptions.

![screenshot of the same Azure Service Health Alert Issue information stored in the Log Analytics Workspace](/assets/10-05-2022-02.png)

At the moment there are multiple ways to send these Azure Service Health Alerts to existing notification systems, being:

* [Service Now](https://learn.microsoft.com/en-us/azure/service-health/service-health-alert-webhook-servicenow)
* [PageDuty](https://learn.microsoft.com/en-us/azure/service-health/service-health-alert-webhook-pagerduty)
* [OpsGenie](https://learn.microsoft.com/en-us/azure/service-health/service-health-alert-webhook-opsgenie)

All of these are using the Azure Action Group *Webhook Action Type*. 

In this blog post I'm going to explain how you can use an **Azure Logic App** as an interface from the Azure Monitor Alert Rule (and Action Group) and JIRA Service Management to have a JIRA Incident created.

Using an Azure Logic App Flow enables you to add logic, like populating the required JIRA Incident properties. E.g. Affected Hardware, Asignment Group etc.

This is specifically helpful when you have added custom fields to the JIRA Incident. The end goal is to have for each of the Azure Service Health Alerts an JIRA Incident being created and assigned to the correct assignment group.

The following JIRA Incident should be created on the above Azure Service Health Issue with title "Cost Management - Duplicate Charges - Mitigated - NK8X-TT8".

![Screenshot JIRA SHA Incident](/assets/10-05-2022-03.png)

Besides the Description field also the fields Affected Hardware and Assignment Group (custom field) are used when creating a JIRA Service Health Alert Incident.

## JIRA Service Management

Jira Service Management is a tool to help you track your tickets, communicate with customers, and resolve requests.

In this example we created a new Custom Field called Assignment Group. You can find more information on how to create this here: [JIRA Assignment Groups](https://confluence.atlassian.com/jira/how-do-i-assign-issues-to-multiple-users-207489749.html?_ga=2.6788850.13189330.1570538441-1940279406.1570033613#HowdoIassignissuestomultipleusers-ManagingIssuesviaGroupOwnership)

In the Logic App workflow we are using both the [Logic App JIRA Connector](https://learn.microsoft.com/en-us/connectors/jira/#create-a-new-issue-(v2)) and JIRA REST API calls.

For the JIRA REST API calls you need to create [JIRA API token](https://support.atlassian.com/atlassian-account/docs/manage-api-tokens-for-your-atlassian-account/). 

This can be used to create a [basic authentication header](https://developer.atlassian.com/cloud/jira/platform/basic-auth-for-rest-apis/#supply-basic-auth-headers) for the JIRA REST API calls.

## Example JIRA REST API calls used in the Logic App Workflow

The following JIRA REST API calls can be used to collect all information neccesary in the Logic App Workflow.

### Get JIRA Fields

```bash
curl -X GET \
   --header 'Authorization: Basic [base64 encoded string]' \
   --header "Content-Type: application/json" \
   'https://[your-domain].atlassian.net/rest/api/3/field' | jq -C
```
This provides you input about any (custom) fields you can use in your Logic App Workflow JIRA REST API calls.

### Get JIRA Issue Types

```bash
curl -X GET \
   --header 'Authorization: Basic [base64 encoded string]' \
   --header "Content-Type: application/json" \
   'https://[your-domain].atlassian.net/rest/api/3/issuetype | jq -C
```

![](/assets/10-05-2022-05.png)

# Log Analytics Query

The following Log Analytics query is being used to retrieve Azure Service Health Alerts.

```sql
// Azure Service Health Issue JIRA Integration Query 
// Date: 22-12-2022
// Version: 1.0
// Description: This query should be use in an Azure Monitor Alert Rule to trigger the Logic App called ServiceHealth Alerts
// Mapping table to link an impacted Azure Service Health Service with the JIRA Assignment Group
// Required JIRA Properties:
// - Description
// - Summary
// - Assignment Group (custom field)
// - Components (make sure you have created Components for each of the Azure Services)
// For an overview of all Azure Service check the Azure Status web page: https://status.azure.com/en-us/status
let Mapping_table = datatable(AZURE_SERVICE: string, JIRA_COMPONENT_NAME: string, JIRA_ASSIGNMENT_GROUP: string)
[
"API Management","Azure","Azure Platform",
"Action Groups","Azure","Azure Platform",
"Activity Logs & Alerts","Azure","Azure Platform",
"Alerts","Azure","Azure Platform",
"Alerts & Metrics","Azure","Azure Platform",
"Application Gateway","Azure","Azure Platform",
"Application Insights","Azure","Azure Platform",
"Automation","Azure","Azure Platform",
"Azure Active Directory","Azure","Azure Platform",
"Azure Active Directory B2C","Azure","Azure Platform",
"Azure Active Directory Domain Services","Azure","Azure Platform",
"Azure Active Directory \\ Enterprise State Roaming","Azure","Azure Platform",
"Azure Bastion","Azure","Azure Platform",
"Azure DDoS Protection","Azure","Azure Platform",
"Azure DNS","Azure","Azure Platform",
"Azure DevOps","Azure","Azure DevOps",
"Azure DevOps \\ Artifacts","Azure","Azure DevOps",
"Azure DevOps \\ Boards","Azure","Azure DevOps",
"Azure DevOps \\ Pipelines","Azure","Azure DevOps",
"Azure DevOps \\ Repos","Azure","Azure DevOps",
"Azure DevOps \\ Test Plans","Azure","Azure DevOps",
"Azure Firewall","Azure","Azure Platform",
"Azure Firewall Manager","Azure","Azure Platform",
"Azure Information Protection","Azure","Azure Platform",
"Azure Migrate","Azure","Azure Platform",
"Azure Red Hat OpenShift","Azure","Azure Platform",
"Azure Reservations","Azure","Azure Platform",
"Azure Search","Azure","Azure Platform",
"Azure Sentinel","Azure","Azure Platform",
"Azure VMware Solution by CloudSimple","Azure","Azure Platform",
"Container Registry","Azure","Azure Platform",
"Data Lake Analytics","Azure","Azure Platform",
"Event Hubs","Azure","Azure Platform",
"ExpressRoute","Azure","Azure Platform",
"ExpressRoute \\ ExpressRoute Circuits","Azure","Azure Platform",
"Key Vault","Azure","Azure Platform",
"Load Balancer","Azure","Azure Platform",
"Media Services \\ Streaming","Azure","Azure Platform",
"Microsoft Azure portal","Azure","Azure Platform",
"Mobile Engagement","Azure","Azure Platform",
"Scheduler","Azure","Azure Platform",
"Stream Analytics","Azure","Azure Platform",
"Time Series Insights","Azure","Azure Platform",
"Traffic Manager","Azure","Azure Platform",
"VPN Gateway","Azure","Azure Platform",
"Virtual Machines","Azure","Azure Platform",
"Cost Management","Azure","Azure Platform"
];
let SHA_table = AzureActivity
| where CategoryValue == "ServiceHealth"
| extend event = parse_json(tostring(parse_json(Properties).eventProperties))
| where event.region in ("Global", "North Europe", "West Europe")
| where ActivityStatusValue == "Active" or ActivityStatusValue == "Resolved"
// All Azure Service Health Events with IncidentType Incident have an impact of 'Service Availability restriced' the rest is None.
| extend IMPACT = case(event.incidentType == "Incident", "Service availability restricted", "None")
| project PROJECT= "ITSMSD", LABELS=CategoryValue, TICKET_ID_Number=tostring(event.trackingId), TICKET_ID_Source= "Azure Monitoring", AZURE_SERVICE=tostring(event.service), IMPACT, Level, ISSUE_TYPE="Incident", SUMMARY_Title=event["title"], SUMMARY_Communication=tostring(event.communication), Status=ActivityStatusValue, impactStartTime=tostring(event.impactStartTime), SubscriptionId, impactedServices=event.impactedServices, TimeGenerated;
SHA_table
| join kind=inner(
    Mapping_table
)
on AZURE_SERVICE
| join kind=inner
( SHA_table
| summarize Subscriptions = make_set(SubscriptionId,1000) by TICKET_ID_Number
) on TICKET_ID_Number
| project-away SubscriptionId, impactedServices
| summarize arg_max(TimeGenerated, *) by TICKET_ID_Number
| sort by impactStartTime desc
| project TICKET_ID_Number, TimeGenerated, Status, AZURE_SERVICE, IMPACT, SUMMARY_Title, SUMMARY_Communication, JIRA_COMPONENT_NAME, JIRA_ASSIGNMENT_GROUP, Subscriptions
```

You need to adjust above query based on your own requirements.

When running above Kusto query within the Log Analytics workspace the following record is being returned.

![Kusto query output results](/assets/12-22-2022-01.png)

If we look at the outputs the following properties are being returned:
- TICKET_ID_Number
- TimeGenerated
- Status
- AZURE_SERVICE
- IMPACT,
- SUMMARY_Title 
- SUMMARY_Communication
- JIRA_COMPONENT_NAME
- JIRA_ASSIGNMENT_GROUP
- Subscriptions

These properties and values will be used in the JIRA REST API calls to create and update JIRA Incidents.

# Log Analytics Workflow

The following Log Analytics Workflows is being used to create and resolve JIRA Incidents for Azure Service Health Alerts

![Logic App Workflow overview screenshot](/assets/12-22-2022-06.png)

Let's get into some of the more detailed **steps** of the Logic App Workflow.

**1. Trigger: When a HTTP request is received**


The workflow is being triggered via a HTTP request triggered via an Azure Monitor Alert Rule and an Action Group.

The first step is that you create an Azure Monitor Action Group that triggers the Logic App Workflow.

![Azure Monitor Action Group configuration screenshot](/assets/12-22-2022-02.png)

Make sure you select as Action type a Logic App and select the earlier created Logic App (workflow). Also use the Common Alert schema in the Action Group configuration.

Now you can create the Azure Monitor Alert which will be used by the Azure Monitor Action Group to trigger the Logic App.

![Azure Monitor Alert Rule configuration screenshot](/assets/12-22-2022-03.png)

Now all the pre-requisites are available to have the Logic App Workflow being triggered when ever the number of records created by the Kusto query are equal to or greater then 1. 

**3. Run query and list results**

This step uses the [Azure Monitor Logs Logic App Connector](https://learn.microsoft.com/en-us/connectors/azuremonitorlogs/#run-query-and-list-results).

![Run query and list results Logic App Connector configuration screenshot](/assets/12-22-2022-04.png)

For the query to be run we are using the Kusto query configured in the Azure Monitor Alert which was passed as a body to the Trigger: When an HTTP Request is received.

```json
body('Parse_JSON')['data']['alertContext']['Condition']['allOf'][0]['searchQuery']
```

**5. For Each - SHA**

In this step we are iterating through the number of Azure Service Health Issues (records returned from the Kusto Query). It could be that there are multiple records being returned.

Within the **For Each - SHA** step there is also a Condition added to check the Status of the Azure Service Health Issue. If the Status is **Active** a **new JIRA Incident** will be created and when the Status is something else, for instance Resolved there will be validations if an existing JIRA Incident can be closed.

For the creation of new JIRA Issue the [Create a new issue (V2) Logic App Connector](https://learn.microsoft.com/en-us/connectors/jira/#create-a-new-issue-(v2)) is used.

![Create new JIRA issue Connector configuration screenshot](/assets/12-22-2022-05.png)

With the properties and values coming from the Kusto query the body of the JIRA Issue REST API is being made.

The body of the Create a new issue (v2) looks as follows:

```json
"body": {
    "fields": {
        "summary": "@{items('For Each - SHA')?['SUMMARY_Title']} - @{items('For Each - SHA')?['TICKET_ID_Number']}",
        "description": "Azure Service Health Issue\n\nStatus: @{items('For Each - SHA')?['Status']} \nStart Time: @{items('For Each - SHA')?['TimeGeneratedUTC']}\nSummary of Impact: @{body('Html to text - Summary Communication')}\nTracking ID: @{items('For Each - SHA')?['TICKET_ID_Number']}\nImpacted Services: @{items('For Each - SHA')?['AZURE_SERVICE']}\nImpacted Subscriptions: @{items('For Each - SHA')?['Subscriptions']}",
        "customfield_10041": "Azure",
        "customfield_10065": "@items('For Each - SHA')?['JIRA_ASSIGNMENT_GROUP']"
    }
}
```

The Custom Field 10041 (Affected Hardware) is used to indicate that the issue is on the Azure Platform and the Custom Field 10065 is used to have a Assignment Group.

When the Status of the Azure Service Health Issue is not Active in the Condition, the right part of the Workflow is being executed.

Step HTTP - Get all Active JIRA SHA Incidents will retrieve all current Azure Service Health JIRA issues where the JIRA status is not equal to Completed.  
  
![HTTP request call to retrieve all JIRA issues screenshot](/assets/12-22-2022-07.png)

If there are any open JIRA issues each of them will be iterated and checked if the TICKED_ID_NUMBER equals the value of an Azure Service Health Issue that has a status of Resolved.

![Foreach loop and validation of JIRA ticket to be closed screenshot](/assets/12-22-2022-08.png)

This logic makes sure that all JIRA Service Health Issues are closed when these are also resolved via the Azure Service Health Issue updates.

Hope this blog post gave you some ideas on how to use Logic Apps to integrate Azure Service Health Issues with JIRA issues.

# FAQ

Here some answer to the Frequently Asked Questions you might have.

1. Why don't you use the out-of-the box [Logic App JIRA Connector](https://learn.microsoft.com/en-us/connectors/jira/#create-a-new-issue-(v2))?
   
   This Connector has limited properties that can be configured to create the JIRA ticket.



# References

* [Azure Service Health Overview](https://azure.microsoft.com/en-gb/get-started/azure-portal/service-health/#overview)
* [Sample Service Health payload](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/activity-log-alerts-webhook)
* [reference for the Jira Service Management Cloud REST APIs](https://developer.atlassian.com/cloud/jira/service-desk/rest/intro/)
* [JIRA REST API call to create an Issue](https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issues/#api-rest-api-3-issue-post)
* [Logic App JIRA Connector](https://learn.microsoft.com/en-us/connectors/jira/#create-a-new-issue-(v2))
* [JIRA Assignment Groups](https://confluence.atlassian.com/jira/how-do-i-assign-issues-to-multiple-users-207489749.html?_ga=2.6788850.13189330.1570538441-1940279406.1570033613#HowdoIassignissuestomultipleusers-ManagingIssuesviaGroupOwnership)
* [Manage API tokens for your Atlassian account](https://support.atlassian.com/atlassian-account/docs/manage-api-tokens-for-your-atlassian-account/)
* [Gist containing the Logic App Code](https://gist.github.com/stefanstranger/342c8a6613faa23a85475184478c47b2)
