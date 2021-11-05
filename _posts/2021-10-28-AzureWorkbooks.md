---
layout: post
title: Using Azure Workbooks for Azure Automation Runbook statistics
categories: [Azure, Statistics]
tags: [Azure, Statistics]
comments: true
---

I recently started to use <a href="https://docs.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-overview" target="_blank">Azure Monitor Workbooks</a> for getting data insights on some of the <a href="https://docs.microsoft.com/en-us/azure/automation/manage-runbooks" target="_blank">Azure Automation Runbooks</a> being used at a customer.

Azure Automation Runbooks are used to create customer environments, like Enteprise-Scale Landing Zones, where DevOps teams can start deploying their Azure Resources to build their solutions on top of.

## High level Overview

The **Azure Automation Runbooks** are triggered from <a href="https://www.servicenow.com/" target="_blank">**Service Now**</a> where DevOps teams can request new Environments and which trigger the creation of a new **Azure DevOps Release** which does all the heavy lifting of the creation of the complete Environment or Landing Zone.

![Process Flow](/assets/EnvironmentRequestFlow.png)

## Azure Monitor Workbook

The goal is to create an Azure Monitor Workbook containing the following information.

![Screenshot Environment Request Workbook](/assets/28-10-2021-2.png)

For these Environment requests we wanted to get more insights in the number and type of environment requests and the time it takes for each of the environments to deploy.

The Diagnostic settings from the Azure Automation Runbook job is used to create the above mentioned insights. Azure Automation can send runbook job status and job streams to your Log Analytics workspace if you configure it.

When looking at the collected Azure Automation Runbook Job Diagnostic Logs we are going to use the following properties.

![Log Analytics query to return Azure Automation Runbook Job information screenshot](/assets/28-10-2021-1.png)

By comparing the Started Runbook Job with the Completed Runbook Job we can calculate the time it took to finish the Runbook and determine the duration of the Environment deployments Runbooks.

But let's first start with an overview of the number of Environment requests per type of environment. 

### Number of Environment Requests per type

The first section of the Workbook shows the number of Environment requests per type. The types defined are Development, Test, Acceptance and Production (DTAP), Sandbox and LandingZone.

We also added Workbook EnvironmentRequests (DTAP, Sandbox and LandingZone) and TimeRange **parameters** to the Workbook for better filtering options.

For the number of Environment Requests per type the following Kusto query is being used in the Workbook.

```kusto
AzureDiagnostics
// filter on Runbookname using EnvironmentRequest parameter
| where RunbookName_s in ({EnvironmentRequest}) or '*' in ({EnvironmentRequest})
// filter on TimeGenerated using the TimeRange Parameter
| where TimeGenerated {TimeRange}
|where ResourceProvider == "MICROSOFT.AUTOMATION"
|where _ResourceId  == "/subscriptions/[subscriptionname]/resourcegroups/[resourcegroupname]/providers/microsoft.automation/automationaccounts/[automationaccountname]"
| where ResultDescription == "Job Completed"
| summarize count() by RunbookName=RunbookName_s
| render columnchart
```

Results

![Colum Chart screenshot](/assets/28-10-2021-3.png)

### Number of Environment Requests per day

The second section of the Workbook shows the number of Enviroment requests per day.

This Kusto query is being used.

```kusto
AzureDiagnostics
| where RunbookName_s in ({EnvironmentRequest}) or '*' in ({EnvironmentRequest})  
| where TimeGenerated {TimeRange}  
| where ResourceProvider == "MICROSOFT.AUTOMATION"  
| where _ResourceId  == "/subscriptions/[subscriptionname]/resourcegroups/[resourcegroupname]/providers/microsoft.automation/automationaccounts/[automationaccountname]"  
| where ResultDescription == "Job Completed"
| summarize count() by RunbookName=RunbookName_s, bin(TimeGenerated, 1d)
| render barchart
```

Results

![Bar chart screenshot](/assets/28-10-2021-4.png)

## Environment Deployment timespans

In the last section of this Workbook we want to have insights into the time it takes for a Runbook to finish.

The following Kusto query is being used to calculate the difference between the Runbook Job with ResultType "Started" and " Completed".

```Kusto
AzureDiagnostics
| where RunbookName_s in ({EnvironmentRequest}) or '*' in ({EnvironmentRequest})
| where TimeGenerated {TimeRange}
| where ResourceProvider == "MICROSOFT.AUTOMATION"
| where _ResourceId  == "/subscriptions/[subscriptionname]/resourcegroups/[resourcegroupname]/providers/microsoft.automation/automationaccounts/[automationaccountname]"  
| where ResultType == "Started" or ResultType == "Completed"
| order by JobId_g, RunbookName_s, TimeGenerated asc
| extend Duration = iff(ResultType != "Started" and prev(ResultType) == "Started", TimeGenerated - prev(TimeGenerated), timespan(null)), SessionNo = prev(RunbookName_s)
| extend StartTime = prev(TimeGenerated)
| where isnotnull(Duration)
| project StartTime, Duration, RunbookName_s
| order by StartTime
| render table
```

Results

![Runbook Duration screenshot](/assets/28-10-2021-5.png)

Here is the complete exported Azure Monitor Runbook as reference:

```json
{
  "version": "Notebook/1.0",
  "items": [
    {
      "type": 1,
      "content": {
        "json": "# Azure Monitor Workbook - Environment Requests\r\n\r\nThis Workbook is providing an overview of the number of Environments being requested by DevOps teams.\r\n\r\nInput for this overview is coming from the Azure Automation Runbooks being triggered by Environment Requests in Service Now Green."
      },
      "name": "Azure Monitor Workbook - Environment Requests"
    },
    {
      "type": 1,
      "content": {
        "json": "## Number of Environment Requests per type"
      },
      "name": "First Section"
    },
    {
      "type": 9,
      "content": {
        "version": "KqlParameterItem/1.0",
        "parameters": [
          {
            "id": "f8fbbbfd-2652-46bf-81ee-c19e08d7e977",
            "version": "KqlParameterItem/1.0",
            "name": "EnvironmentRequest",
            "label": "Environment Request",
            "type": 2,
            "description": "Filter on Environment request types",
            "multiSelect": true,
            "quote": "'",
            "delimiter": ",",
            "query": "AzureDiagnostics\r\n| where ResourceProvider == \"MICROSOFT.AUTOMATION\"\r\n| where _ResourceId  == \"subscriptions/[subscriptionname]/resourcegroups/[resourcegroupname]/providers/microsoft.automation/automationaccounts/[automationaccountname]\"\r\n| where ResultDescription == \"Job Completed\"\r\n| distinct RunbookName_s\r\n| project RunbookName = RunbookName_s",
            "crossComponentResources": [
              "/subscriptions/[subscriptiond]resourceGroups/[resourcegroupname]/providers/Microsoft.OperationalInsights/workspaces/[workspacename]"
            ],
            "typeSettings": {
              "additionalResourceOptions": [
                "value::all"
              ],
              "showDefault": false
            },
            "timeContext": {
              "durationMs": 2592000000
            },
            "defaultValue": "value::all",
            "queryType": 0,
            "resourceType": "microsoft.operationalinsights/workspaces",
            "value": [
              "Create-DTAP"
            ]
          },
          {
            "id": "3d5c9981-4dc2-43ac-b6bb-8d177cd9d0bf",
            "version": "KqlParameterItem/1.0",
            "name": "TimeRange",
            "type": 4,
            "isRequired": true,
            "value": {
              "durationMs": 1209600000
            },
            "typeSettings": {
              "selectableValues": [
                {
                  "durationMs": 3600000
                },
                {
                  "durationMs": 14400000
                },
                {
                  "durationMs": 43200000
                },
                {
                  "durationMs": 86400000
                },
                {
                  "durationMs": 172800000
                },
                {
                  "durationMs": 259200000
                },
                {
                  "durationMs": 604800000
               },
                {
                  "durationMs": 1209600000
                },
                {
                  "durationMs": 2419200000
                },
                {
                  "durationMs": 2592000000
                },
                {
                  "durationMs": 5184000000
                },
                {
                  "durationMs": 7776000000
                }
              ],
              "allowCustom": true
            },
            "label": "Time Range"
          }
        ],
        "style": "pills",
        "queryType": 0,
        "resourceType": "microsoft.operationalinsights/workspaces"
      },
      "name": "parameters - 2"
    },
    {
      "type": 3,
      "content": {
        "version": "KqlItem/1.0",
        "query": "AzureDiagnostics\r\n| where RunbookName_s in ({EnvironmentRequest}) or '*' in ({EnvironmentRequest})\r\n| where TimeGenerated {TimeRange}\r\n| where ResourceProvider == \"MICROSOFT.AUTOMATION\"\r\n| where _ResourceId  == \"/subscriptions/[subscriptionname]/resourcegroups/[resourcegroupname]/providers/microsoft.automation/automationaccounts/[automationaccountname]\"\r\n| where ResultDescription == \"Job Completed\"\r\n| summarize count() by RunbookName=RunbookName_s\r\n| render columnchart",
        "size": 0,
        "queryType": 0,
        "resourceType": "microsoft.operationalinsights/workspaces",
        "crossComponentResources": [
          "/subscriptions/[subscriptiond]resourceGroups/[resourcegroupname]/providers/Microsoft.OperationalInsights/workspaces/[workspacename]"
        ],
        "visualization": "barchart"
      },
      "name": "query - 0"
    },
    {
      "type": 1,
      "content": {
        "json": "## Number of Environment Requests per day"
      },
      "name": "Section 2"
    },
    {
      "type": 3,
      "content": {
        "version": "KqlItem/1.0",
        "query": "AzureDiagnostics\r\n| where RunbookName_s in ({EnvironmentRequest}) or '*' in ({EnvironmentRequest})\r\n| where TimeGenerated {TimeRange}\r\n| where ResourceProvider == \"MICROSOFT.AUTOMATION\"\r\n| where _ResourceId  == \"subscriptions/[subscriptionname]/resourcegroups/[resourcegroupname]/providers/microsoft.automation/automationaccounts/[automationaccountname]\"\r\n| where ResultDescription == \"Job Completed\"\r\n| summarize count() by RunbookName=RunbookName_s, bin(TimeGenerated, 1d)\r\n| render barchart",
        "size": 0,
        "queryType": 0,
        "resourceType": "microsoft.operationalinsights/workspaces",
        "crossComponentResources": [
          "/subscriptions/[subscriptiond]resourceGroups/[resourcegroupname]/providers/Microsoft.OperationalInsights/workspaces/[workspacename]"
        ],
        "visualization": "linechart",
        "graphSettings": {
          "type": 0,
          "topContent": {
            "columnMatch": "RunbookName",
            "formatter": 1
          },
          "centerContent": {
            "columnMatch": "count_",
            "formatter": 1,
            "numberFormat": {
              "unit": 17,
              "options": {
                "maximumSignificantDigits": 3,
                "maximumFractionDigits": 2
              }
            }
          },
          "nodeIdField": "RunbookName",
          "sourceIdField": "RunbookName",
          "targetIdField": "RunbookName",
          "graphOrientation": 3,
          "showOrientationToggles": false,
          "nodeSize": null,
          "staticNodeSize": 100,
          "colorSettings": null,
          "hivesMargin": 5
        }
      },
      "name": "query - 3"
    },
    {
      "type": 1,
      "content": {
        "json": "## Environment Deployment timespans. \r\n\r\nHow long does it takes for each deployment to finish?"
      },
      "name": "text - 6"
    },
    {
      "type": 3,
      "content": {
        "version": "KqlItem/1.0",
        "query": "// Environment Deployment speeds\r\nAzureDiagnostics\r\n| where RunbookName_s in ({EnvironmentRequest}) or '*' in ({EnvironmentRequest})\r\n| where TimeGenerated {TimeRange}\r\n| where ResourceProvider == \"MICROSOFT.AUTOMATION\"\r\n| where _ResourceId  == \"subscriptions/[subscriptionname]/resourcegroups/[resourcegroupname]/providers/microsoft.automation/automationaccounts/[automationaccountname]\"\r\n| where ResultType == \"Started\" or ResultType == \"Completed\"\r\n| order by JobId_g, RunbookName_s, TimeGenerated asc\r\n| extend Duration = iff(ResultType != \"Started\" and prev(ResultType) == \"Started\", TimeGenerated - prev(TimeGenerated), timespan(null)), SessionNo = prev(RunbookName_s)\r\n| extend StartTime = prev(TimeGenerated)\r\n| where isnotnull(Duration)\r\n| project StartTime, Duration, RunbookName_s\r\n| order by StartTime\r\n| render table",
        "size": 0,
        "queryType": 0,
        "resourceType": "microsoft.operationalinsights/workspaces",
        "crossComponentResources": [
          "/subscriptions/[subscriptiond]resourceGroups/[resourcegroupname]/providers/Microsoft.OperationalInsights/workspaces/[workspacename]"
        ]
      },
      "name": "query - 7"
    }
  ],
  "fallbackResourceIds": [
    "Azure Monitor"
  ],
  "$schema": "https://github.com/Microsoft/Application-Insights-Workbooks/blob/master/schema/workbook.json"
}
```

Hope you found this usefull information for creating your own Azure Monitor Workbooks.

# References

- [Azure Monitor Runbooks](https://docs.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-overview)
- [Azure Automation Runbooks](https://docs.microsoft.com/en-us/azure/automation/manage-runbooks)
- [Service Now](https://www.servicenow.com/)
- [ Microsoft Docs - Forward Azure Automation job data to Azure Monitor logs](https://docs.microsoft.com/en-us/azure/automation/automation-manage-send-joblogs-log-analytics)
- [Microsoft Docs - Workbook parameters](https://docs.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-parameters)