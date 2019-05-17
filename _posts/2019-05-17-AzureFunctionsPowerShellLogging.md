---
layout: post
title: Logging in PowerShell for Azure functions
categories: [Azure, Functions, PowerShell]
tags: [Azure, Functions, PowerShell]
comments: true
---

While working on a PowerShell Azure Function for my PowerShell Conference Europe 2019 session I wanted to enable some logging and it was not as straight forward as I initially thought.

## Logging

Logging in PowerShell functions works like regular PowerShell logging. You can use the logging cmdlets to write to each output stream. Each cmdlet maps to a log level used by Functions.


| Functions logging level | Logging cmdlet     |     |                               |
|-------------------------|--------------------|-------------|--------------------------------------|
| Error                   | Write-Error        |             |                                      |
| Warning                 | Write-Warning      |             |                                      |
| Information             | Write-Information  | Information | Writes to Information level logging. |
|                         | Write-Host         |             |                                      |
|                         | Write-Output       |             |                                      |
| Debug                   | Write-Debug        |             |                                      |
| Trace                   | Write-Progress     |             |                                      |
|                         | Write-Verbose      |             |                                      |

In addition to these cmdlets, anything written to the pipeline is redirected to the Information log level and displayed with the default PowerShell formatting.

|  Important |
|----------|
| Using the Write-Verbose or Write-Debug cmdlets is not enough to see verbose and debug level logging. You must also configure the log level threshold, which declares what level of logs you actually care about. To learn more, see Configure the function app log level. |

### Configure the function app log level

Functions lets you define the threshold level to make it easy to control the way Functions writes to the logs. To set the threshold for all traces written to the console, use the logging.logLevel.default property in the host.json file. This setting applies to all functions in your function app.

The following example sets the threshold to enable verbose logging for all functions, but sets the threshold to enable debug logging for a function named MyFunction:

```json
{
    "logging": {
        "logLevel": {
            "Function.MyFunction": "Debug",
            "default": "Trace"
        }
    }
}
```

All above information is <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-powershell#logging" target="_blank">available on Microsoft Docs</a>.

But what is needed in your run.ps1 script?

You need to add the Verbose switch to your Write-Verbose cmdlet.

Example:
```powershell
# Get TriggerMetadata
Write-Verbose ($TriggerMetadata | Convertto-Json) -Verbose
```

![VerboseLogging](/assets/2019-05-17.png)