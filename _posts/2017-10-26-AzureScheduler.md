---
layout: post
title: Azure Scheduler - Schedule your Runbooks more often than every hour
categories: [Azure, Automation, Schedule]
tags: [Azure, Automation, Schedule]
comments: true
---
Have you ever wanted to schedule your Azure Automation Runbook every 15 minutes?

**Scenario**

Suppose you have a Runbook that you want to run more often than every hour, say every 15 minutes. This is not possible to configure with the default Schedule Asset in Azure Automation.
You only have the options for a recurring schedule for Hour, Day, Week, Month.

![]/assets/(AzureSchedule.png)

**Azure Scheduler**

With Azure Scheduler you can invoke actions such as calling HTTP/S endpoints or posting a message to a storage queue on any schedule. With Microsoft Azure Scheduler, you create jobs in the cloud that reliably call services both inside and outside of Microsoft Azure and run those jobs on demand or on a regularly recurring schedule, or designate them for a future date. 
This service is currently available as a standalone API.
Use Scheduler to:
* Invoke a Web Service over HTTP/s
* Post a message to an Azure Storage Queue
* Post a message to an Azure Service Bus Queue
* Post a message to an Azure Service Bus Topic

**Webhook**

Before you can trigger your Runbook via the Azure Schedule you first need to add a Webhook to your Runbook.

Check my blog post "<a href="https://blogs.technet.microsoft.com/stefan_stranger/2017/03/18/azure-automation-runbook-webhook-lesson-learned/" target="_blank">Azure Automation Runbook Webhook lesson learned</a>" for more information.

**Configure Azure Schedule**

Go to Azure Portal and create a new Azure Schedule.

Next create a new Job Collection. Make sure you don't select the Free Tier because this has a max frequency of 1 hour.
![](/assets/JobCollectionTier.png)

Configure the Action Settings
![](/assets/ActionSettingsNew.png)

Configure if needed headers and Optional settings. Don't forget to also configure the ContentType Header if you use a body in your POST Request.

![](/assets/OptionalSettingsNew.png)

And finally configure the Schedule.
![](/assets/ScheduleSettings.png)

Go run your Azure Automation Runbooks more often! :-)