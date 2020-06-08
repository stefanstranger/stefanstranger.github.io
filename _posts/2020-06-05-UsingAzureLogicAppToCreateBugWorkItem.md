---
layout: post
title: Using Azure Logic Apps to create an Azure DevOps Bug WorkItem
categories: [CI/CD, Azure]
tags: [CI/CD, Azure]
comments: true
---

# Table of contents

- [Table of contents](#table-of-contents)
- [Azure Logic Apps](#azure-logic-apps)
  - [Trigger](#trigger)
  - [Azure DevOps Service Hook](#azure-devops-service-hook)
  - [Configuration of Logic App](#configuration-of-logic-app)
  - [Configuration of Azure DevOps Web Hook](#configuration-of-azure-devops-web-hook)
  - [Add Logic App Actions](#add-logic-app-actions)
  - [Update Azure DevOps Web Hook](#update-azure-devops-web-hook)
- [Trigger Web hook and Logic App in failed Pipeline](#trigger-web-hook-and-logic-app-in-failed-pipeline)
- [Logic App Code View](#logic-app-code-view)

Some time ago I created a private Azure DevOps Extension for a customer which would create an Azure DevOps Bug WorkItem when a task in a Release would fail.

When returning at that customer this Extension was still used but it didn't work for (yaml) build pipelines. Instead of updating this Extension I investigated if I was able to use Azure DevOps Service Hooks to trigger an Azure Logic App which would create a Bug WorkItem for failed (build) Yaml Pipelines.

After sharing a successful test implementation result on <a href="https://twitter.com/sstranger/status/1267085461706747905" target="_blank">Twitter</a> it seemed that some more people are interested in how I got this working.

![Screenshot Twitter communication](/assets/2020-06-05-01.png)

So here is that blog post :-)

# Azure Logic Apps

For those unfamiliar with Logic Apps, it is a cloud service that helps you schedule, automate, and orchestrate tasks, business processes, and workflows when you need to integrate apps, data, systems, and services across enterprises or organizations. 

Logic Apps simplifies how you design and build scalable solutions for app integration, data integration, system integration, enterprise application integration (EAI), and business-to-business (B2B) communication, whether in the cloud, on premises, or both. You can find more information <a href="https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-overview" target="_blank">here</a>.

## Trigger

For creating the Azure DevOps Bug WorkItem we need a trigger, which fires when a specific event (like a failed task in a build pipeline) happens. Each time that the trigger fires, the Logic Apps engine creates a logic app instance that runs the actions in the workflow.

For our Logic App we are going to use a Response <a href="https://docs.microsoft.com/en-us/azure/connectors/connectors-native-reqres#add-a-response-action" target="_blank">Request trigger</a>.

This Trigger will be run by the Azure DevOps Service Hooks Web Hook.

## Azure DevOps Service Hook

<a href="https://docs.microsoft.com/en-us/azure/devops/service-hooks/services/webhooks?view=azure-devops" target="_blank">Web Hooks</a> provides a way to send a JSON representation of an event to any service. All that is required is a public endpoint (HTTP or HTTPS).


![screenshot of Web Hook in Azure DevOps](/assets/2020-06-05-02.png)

Later in this blog post I'll explain the configuration needed here to trigger the Logic App. Let's now first start with the creation of the Logic App.

## Configuration of Logic App

Go to the Azure Portal and create a new Logic App Resource

![Screenshot Azure Portal Logic App Creation](/assets/2020-06-05-03.png)

You should now have an empty Logic App Resource created. 

![Screenshot Logic App in Azure Portal](/assets/2020-06-05-04.png)

Let's add our first Action "When a HTTP request is received"  by opening the Designer.

![Screenshot HTTP request](/assets/2020-06-05-05.png)
 
At a new POST Method Parameter and save the Logic App and copy the HTTP Get URL.

![Screenshot HTTP Request Post Method](/assets/2020-06-05-06.png)

We need to use this URL in the Azure DevOps Web Hook later.

## Configuration of Azure DevOps Web Hook

Open Azure DevOps Project Settings and create a new Web Hook using the URL from previous step.

![Screenshot Azure DevOps Web Hook Configuration](/assets/2020-06-05-07.png)

Click on next after selecting Web Hooks.

We want for each of the completed Builds with Status Failed to have the Logic App being triggered.

![Screenshot of Web Hook settings](/assets/2020-06-05-08.png)

Configure the properties of the Web Hook and test the webhook.

![Screenshot Web Hook Properties](/assets/2020-06-05-09.png)

Copy the Request from the Test to the Logic App to be used as sample payload to generate schema.

![Screenshot of the Request](/assets/2020-06-05-10.png)

![Sceenshot of sample payload to generate schema in Logic Apps](/assets/2020-06-05-11.png)

![Sceenshot of sample payload to generate schema in Logic Apps](/assets/2020-06-05-12.png)

Click on Done and save Logic App.

## Add Logic App Actions

The following Actions need to be added.

![Screenshot of the Actions in Logic Apps](/assets/2020-06-05-13.png)

1. Initialize variable - Iteration

![Screenshot Initialize Iteration variable](/assets/2020-06-05-14.png)

Save Logic App.

2. Initialize - Organization

![Screenshot of Initialize Organization Variable](/assets/2020-06-05-15.png)

Save Logic App.

3. Set variable - Organization

![Screenshot of Set Organization Variable](/assets/2020-06-05-16.png)

![Screenshot of Initialize Organization Variable Expression](/assets/2020-06-05-17.png)

Expression:

```Batch
replace(split(triggerBody()?['resource']?['url'],'.')[0],'https://','')
```

**Explanation:**

From the "When a HTTP request is received" output we want to extract the Organization name and store that value in a variable.

![Screenshot of When a HTTP request is received output](/assets/2020-0605-18.png)

Save Logic App.

4. Send a HTTP request to Azure DevOps - Get Iteration

With this action we want to retrieve the current Azure DevOps Iteration. More info on this REST API can be found <a href="https://docs.microsoft.com/en-us/rest/api/azure/devops/work/Teamsettings/Get?view=azure-devops-rest-5.1" target="_blank">here</a>.

![Screenshot Send an HTTP request to Azure DevOps to get Iteration](/assets/2020-06-05-19.png)

You first need to create a connection to Azure DevOps.

![Screenshot to Sign in to Azure DevOps Connection](/assets/2020-06-05-20.png)

Configure the Send an HTTP Request to Azure DevOps without configuring the Relative URL.

![Screenshot to configure Send an HTTP request to Azure DevOps](/assets/2020-06-05-21.png)

Go to Code View after first saving the Logic App.

![Screenshot of Code View of Logic App](/assets/2020-06-05-22.png)

```Batch
https://dev.azure.com/@{variables('Organization')}/@{triggerOutputs()['queries']['Project']}/@{triggerOutputs()['queries']['Team']}/_apis/work/teamsettings?api-version=5.1
```

**Explanation:**

The URL should look something like this:
```Batch
https://dev.azure.com/stefanstranger/Contoso/Contoso Team/_apis/work/teamsettings?api-version=5.1
```

| Expression| Value | Explanation |
|----------|----------|----------|
| @{variables('Organization')} | stefanstranger | This value is from the earlier variable|
| @{triggerOutputs()['queries']['Project']} | Contoso | This value is coming from a parameter input on the Web Hook Webrequest. Example: https://foo.com?Project=Contoso |
| @{triggerOutputs()['queries']['Team']} | Contoso Team | This value is coming from a parameter input on the Web Hook Webrequest. Example: https://foo.com?Team=Contoso%20Team |  

![](/assets/2020-06-05-23.png)

Save Logic App.

5. Set Variable - Iteration

![Screenshot of Set Iteration Variable Action](/assets/2020-06-05-24.png)

Use the following Dynamic content. We want to use the DefaultIteration path from the previous Action.

```Batch
"@body('Send_an_HTTP_request_to_Azure_DevOps_-_Get_Iteration')['DefaultIteration']['path']"
```

Got to the Designer add temp value and save Action before going to View Code.

![Screenshot of the Designer](/assets/2020-06-05-25.png)

![Screenshot of the Code View](/assets/2020-06-05-26.png)

![Screenshot of the Code View for the Set_Variable](/assets/2020-06-05-27.png)

Save Logic App.

6. Create a Work Item

Add a new Azure DevOps Action to your Logic App.

![Screenshot of Create a Work Item Action](/assets/2020-06-05-28.png)

Configure the following properties:
* Organization Name, 
* Project Name 
* WorkItem Type
* Title
* Description
* Area Path
* Iteration Path
* Repro Steps

![Screenshot of temp parameters for Create a Work Item Action](/assets/2020-06-05-29.png)

Save Action and open Code View Editor and replace with the following:

```Batch
"Create_a_work_item": {
                "inputs": {
                    "body": {
                        "area": "@{triggerOutputs()['queries']['Area']}",
                        "description": "@{triggerBody()?['detailedMessage'].text}",
                        "dynamicFields": {
                            "Microsoft.VSTS.TCM.ReproSteps": "@{triggerBody()?['detailedMessage'].html}"
                        },
                        "iteration": "@{triggerOutputs()['queries']['Area']}@{variables('Iteration')}",
                        "title": "@{triggerBody()?['message']?['text']} - @{triggerBody()?['resource']?['finishTime']}"
                    },
                    "host": {
                        "connection": {
                            "name": "@parameters('$connections')['visualstudioteamservices']['connectionId']"
                        }
                    },
                    "method": "patch",
                    "path": "/@{encodeURIComponent('Contoso')}/_apis/wit/workitems/$Bug",
                    "queries": {
                        "account": "stefanstranger"
                    }
                },
                "runAfter": {
                    "Set_variable_-_Iteration": [
                        "Succeeded"
                    ]
                },
                "type": "ApiConnection"
            },
```

![Screenshot of Code View of Create a Work Item Action](/assets/2020-06-05-30.png)

Save Logic App.

## Update Azure DevOps Web Hook

We now need to add the following parameter to the web hook url:
* Area
* Project
* Team

Open the configured Web Hook in Azure DevOps and copy the content to Notepad and add the following:

![Screenshot of Azure DevOp Web Hook test](/assets/2020-06-05-31.png)

Test the updated Web Hook in both Azure DevOps and in Azure.

![Screenshot of Azure DevOps Web Hook test](/assets/2020-06-05-32.png)

If you get an unauthorized error message on the Send an HTTP request to Azure DevOps Action.

![Screenshot of Unauthorized Error in Logic App](/assets/2020-06-05-33.png)

Got to API Connections and Delete the Azure DevOps Connection.

![Screenshot of API Connection setting in Azure Portal](/assets/2020-06-05-34.png)

Go to the Logic Analytics Designer and re-add the Connection by Adding a new one.

![Screenshot of adding API Connection](/assets/2020-06-05-35.png)


Save the Logic App. And Authorize API Connection again and save.

![Screenshot or Authorizing the API Connection](/assets/2020-06-05-36.png)


# Trigger Web hook and Logic App in failed Pipeline

The final test is when we create a (build) pipeline which fails to validate the Logic App being triggered and the Bug Work Item being created.

yaml pipeline code:

```yaml
trigger:
  - none

steps:
  - powershell: |
      throw 'Something failed!'
```

Trigger the pipeline.

![Screenshot of Azure DevOps Pipeline](/assets/2020-06-05-37.png)

Check the Logic App Runs History in the Portal.

![Screenshot of Logic App Runs History](/assets/2020-06-0538.png)

And finally check in Azure DevOps if the Bug Work Item is being created.

![Screenshot of Azure DevOps Bug Work Item creation](/assets/2020-06-05-39.png)


# Logic App Code View

Here is the complete Code for the Logic App.

```json
{
    "definition": {
        "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
        "actions": {
            "Create_a_work_item": {
                "inputs": {
                    "body": {
                        "area": "@{triggerOutputs()['queries']['Area']}",
                        "description": "@{triggerBody()?['detailedMessage'].text}",
                        "dynamicFields": {
                            "Microsoft.VSTS.TCM.ReproSteps": "@{triggerBody()?['detailedMessage'].html}"
                        },
                        "iteration": "@{triggerOutputs()['queries']['Area']}@{variables('Iteration')}",
                        "title": "@{triggerBody()?['message']?['text']} - @{triggerBody()?['resource']?['finishTime']}"
                    },
                    "host": {
                        "connection": {
                            "name": "@parameters('$connections')['visualstudioteamservices']['connectionId']"
                        }
                    },
                    "method": "patch",
                    "path": "/@{encodeURIComponent('Contoso')}/_apis/wit/workitems/$Bug",
                    "queries": {
                        "account": "contosodemo"
                    }
                },
                "runAfter": {
                    "Set_variable_-_Iteration": [
                        "Succeeded"
                    ]
                },
                "type": "ApiConnection"
            },
            "Initialize_variable_-_Iteration": {
                "inputs": {
                    "variables": [
                        {
                            "name": "Iteration",
                            "type": "string"
                        }
                    ]
                },
                "runAfter": {},
                "type": "InitializeVariable"
            },
            "Initialize_variable_-_Organization": {
                "inputs": {
                    "variables": [
                        {
                            "name": "Organization",
                            "type": "string"
                        }
                    ]
                },
                "runAfter": {
                    "Initialize_variable_-_Iteration": [
                        "Succeeded"
                    ]
                },
                "type": "InitializeVariable"
            },
            "Send_an_HTTP_request_to_Azure_DevOps_-_Get_Iteration": {
                "inputs": {
                    "body": {
                        "Method": "GET",
                        "Uri": "https://dev.azure.com/@{variables('Organization')}/@{triggerOutputs()['queries']['Project']}/@{triggerOutputs()['queries']['Team']}/_apis/work/teamsettings?api-version=5.1"
                    },
                    "host": {
                        "connection": {
                            "name": "@parameters('$connections')['visualstudioteamservices']['connectionId']"
                        }
                    },
                    "method": "post",
                    "path": "/httprequest",
                    "queries": {
                        "account": "contosodemo"
                    }
                },
                "runAfter": {
                    "Set_variable_-_Organization": [
                        "Succeeded"
                    ]
                },
                "type": "ApiConnection"
            },
            "Set_variable_-_Iteration": {
                "inputs": {
                    "name": "Iteration",
                    "value": "@body('Send_an_HTTP_request_to_Azure_DevOps_-_Get_Iteration')['DefaultIteration']['path']"
                },
                "runAfter": {
                    "Send_an_HTTP_request_to_Azure_DevOps_-_Get_Iteration": [
                        "Succeeded"
                    ]
                },
                "type": "SetVariable"
            },
            "Set_variable_-_Organization": {
                "inputs": {
                    "name": "Organization",
                    "value": "@{replace(split(triggerBody()?['resource']?['url'],'.')[0],'https://','')}"
                },
                "runAfter": {
                    "Initialize_variable_-_Organization": [
                        "Succeeded"
                    ]
                },
                "type": "SetVariable"
            }
        },
        "contentVersion": "1.0.0.0",
        "outputs": {},
        "parameters": {
            "$connections": {
                "defaultValue": {},
                "type": "Object"
            }
        },
        "triggers": {
            "manual": {
                "inputs": {
                    "method": "POST",
                    "schema": {
                        "properties": {
                            "createdDate": {
                                "type": "string"
                            },
                            "detailedMessage": {
                                "properties": {
                                    "html": {
                                        "type": "string"
                                    },
                                    "markdown": {
                                        "type": "string"
                                    },
                                    "text": {
                                        "type": "string"
                                    }
                                },
                                "type": "object"
                            },
                            "eventType": {
                                "type": "string"
                            },
                            "id": {
                                "type": "string"
                            },
                            "message": {
                                "properties": {
                                    "html": {
                                        "type": "string"
                                    },
                                    "markdown": {
                                        "type": "string"
                                    },
                                    "text": {
                                        "type": "string"
                                    }
                                },
                                "type": "object"
                            },
                            "notificationId": {
                                "type": "integer"
                            },
                            "publisherId": {
                                "type": "string"
                            },
                            "resource": {
                                "properties": {
                                    "buildNumber": {
                                        "type": "string"
                                    },
                                    "definition": {
                                        "properties": {
                                            "batchSize": {
                                                "type": "integer"
                                            },
                                            "definitionType": {
                                                "type": "string"
                                            },
                                            "id": {
                                                "type": "integer"
                                            },
                                            "name": {
                                                "type": "string"
                                            },
                                            "triggerType": {
                                                "type": "string"
                                            },
                                            "url": {
                                                "type": "string"
                                            }
                                        },
                                        "type": "object"
                                    },
                                    "drop": {
                                        "properties": {
                                            "downloadUrl": {
                                                "type": "string"
                                            },
                                            "location": {
                                                "type": "string"
                                            },
                                            "type": {
                                                "type": "string"
                                            },
                                            "url": {
                                                "type": "string"
                                            }
                                        },
                                        "type": "object"
                                    },
                                    "dropLocation": {
                                        "type": "string"
                                    },
                                    "finishTime": {
                                        "type": "string"
                                    },
                                    "hasDiagnostics": {
                                        "type": "boolean"
                                    },
                                    "id": {
                                        "type": "integer"
                                    },
                                    "lastChangedBy": {
                                        "properties": {
                                            "displayName": {
                                                "type": "string"
                                            },
                                            "id": {
                                                "type": "string"
                                            },
                                            "imageUrl": {
                                                "type": "string"
                                            },
                                            "uniqueName": {
                                                "type": "string"
                                            },
                                            "url": {
                                                "type": "string"
                                            }
                                        },
                                        "type": "object"
                                    },
                                    "log": {
                                        "properties": {
                                            "downloadUrl": {
                                                "type": "string"
                                            },
                                            "type": {
                                                "type": "string"
                                            },
                                            "url": {
                                                "type": "string"
                                            }
                                        },
                                        "type": "object"
                                    },
                                    "queue": {
                                        "properties": {
                                            "id": {
                                                "type": "integer"
                                            },
                                            "name": {
                                                "type": "string"
                                            },
                                            "queueType": {
                                                "type": "string"
                                            },
                                            "url": {
                                                "type": "string"
                                            }
                                        },
                                        "type": "object"
                                    },
                                    "reason": {
                                        "type": "string"
                                    },
                                    "requests": {
                                        "items": {
                                            "properties": {
                                                "id": {
                                                    "type": "integer"
                                                },
                                                "requestedFor": {
                                                    "properties": {
                                                        "displayName": {
                                                            "type": "string"
                                                        },
                                                        "id": {
                                                            "type": "string"
                                                        },
                                                        "imageUrl": {
                                                            "type": "string"
                                                        },
                                                        "uniqueName": {
                                                            "type": "string"
                                                        },
                                                        "url": {
                                                            "type": "string"
                                                        }
                                                    },
                                                    "type": "object"
                                                },
                                                "url": {
                                                    "type": "string"
                                                }
                                            },
                                            "required": [
                                                "id",
                                                "url",
                                                "requestedFor"
                                            ],
                                            "type": "object"
                                        },
                                        "type": "array"
                                    },
                                    "retainIndefinitely": {
                                        "type": "boolean"
                                    },
                                    "sourceGetVersion": {
                                        "type": "string"
                                    },
                                    "startTime": {
                                        "type": "string"
                                    },
                                    "status": {
                                        "type": "string"
                                    },
                                    "uri": {
                                        "type": "string"
                                    },
                                    "url": {
                                        "type": "string"
                                    }
                                },
                                "type": "object"
                            },
                            "resourceContainers": {
                                "properties": {
                                    "account": {
                                        "properties": {
                                            "id": {
                                                "type": "string"
                                            }
                                        },
                                        "type": "object"
                                    },
                                    "collection": {
                                        "properties": {
                                            "id": {
                                                "type": "string"
                                            }
                                        },
                                        "type": "object"
                                    },
                                    "project": {
                                        "properties": {
                                            "id": {
                                                "type": "string"
                                            }
                                        },
                                        "type": "object"
                                    }
                                },
                                "type": "object"
                            },
                            "resourceVersion": {
                                "type": "string"
                            },
                            "subscriptionId": {
                                "type": "string"
                            }
                        },
                        "type": "object"
                    }
                },
                "kind": "Http",
                "type": "Request"
            }
        }
    },
    "parameters": {
        "$connections": {
            "value": {
                "visualstudioteamservices": {
                    "connectionId": "/subscriptions/13bd0bca-5bf3-4c8b-be25-3661251884b8/resourceGroups/bug-workitem-demo-rg/providers/Microsoft.Web/connections/visualstudioteamservices",
                    "connectionName": "visualstudioteamservices",
                    "id": "/subscriptions/13bd0bca-5bf3-4c8b-be25-3661251884b8/providers/Microsoft.Web/locations/westeurope/managedApis/visualstudioteamservices"
                }
            }
        }
    }
}
```