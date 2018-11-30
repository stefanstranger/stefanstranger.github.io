---
layout: post
title: PowerShell Azure Functions lessons learned
date: 2017-01-29 16:34
author: stefan stranger
comments: true
categories: [Azure, Functions, PowerShell]
---
<p>Lately I’m playing with PowerShell Azure Functions and I learned there are some things I needed to learn and that’s why I want to share some more info about this topic.</p> <p><strong>What are Azure Functions?</strong></p> <p>Azure Functions is an event driven, compute-on-demand experience that extends the existing Azure application platform with capabilities to implement code triggered by events occurring in virtually any Azure or 3rd party service as well as on-premises systems. Azure Functions allows developers to take action by connecting to data sources or messaging solutions, thus making it easy to process and react to events. Azure Functions scale based on demand and you pay only for the resources you consume. (<a href="https://github.com/Azure/Azure-Functions/" target="_blank">from Github)</a></p> <p>On the Github page about Azure Functions you can find all the info to <a href="https://functions.azure.com/" target="_blank">get started.</a></p> <p><strong>PowerShell Azure Functions</strong></p> <p>Azure Functions enables you to create scheduled or triggered units of code implemented in various programming languages. PowerShell is one of those programming languages.</p> <p>A good starting point is a blog post from <a href="https://david-obrien.net/2016/07/azure-functions-PowerShell/" target="_blank">David O’Brien Azure Functions – PowerShell</a>. </p> <p>If you look at an example from David you see a special variable being used ‘$res’. At first it was not clear to me where this variable was being defined. To find out more I started to create a Function using the Get-Variable cmdlet.</p> <p>Use the following PowerShell script in your Function to retrieve the automatic variables available in Azure PowerShell functions.</p> <div id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:1374e85e-7abf-49de-8b93-be37f069cdd5" class="wlWriterEditableSmartContent" style="float: none;padding-bottom: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px"><pre>
[sourcecode language='powershell'  padlinenumbers='true']
write-Output &#039;Getting variables&#039;
$result = Get-Variable | out-string
[/sourcecode]
</pre>
</div>
<p>If you run this in your Azure PowerShell function you will see the following in your logs section.</p>
<div id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:f5f1c948-38b2-4b89-9b47-dc0581465470" class="wlWriterEditableSmartContent" style="float: none;padding-bottom: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px"><pre>
[sourcecode language='powershell' ]
2017-01-29T16:01:51.538 Function started (Id=6528a25c-61cc-4d16-90ad-4869e47dc599)
2017-01-29T16:01:51.867 Getting variables
2017-01-29T16:01:51.898 Name                           Value                                            
----                           -----                                            
$                                                                               
?                              True                                             
^                                                                               
args                           {}                                               
ConfirmPreference              High                                             
ConsoleFileName                                                                 
DebugPreference                SilentlyContinue                                 
Error                          {}                                               
ErrorActionPreference          Continue                                         
ErrorView                      NormalView                                       
ExecutionContext               System.Management.Automation.EngineIntrinsics    
false                          False                                            
FormatEnumerationLimit         4                                                
HOME                                                                            
Host                           System.Management.Automation.Internal.Host.Int...
input                          System.Collections.ArrayList+ArrayListEnumerat...
InvocationId                   6528a25c-61cc-4d16-90ad-4869e47dc599             
MaximumAliasCount              4096                                             
MaximumDriveCount              4096                                             
MaximumErrorCount              256                                              
MaximumFunctionCount           4096                                             
MaximumHistoryCount            4096                                             
MaximumVariableCount           4096                                             
MyInvocation                   System.Management.Automation.InvocationInfo      
NestedPromptLevel              0                                                
null                                                                            
OutputEncoding                 System.Text.ASCIIEncoding                        
outputFile                     D:\local\Temp\Functions\Binding\6528a25c-61cc-...
PID                            9936                                             
ProgressPreference             Continue                                         
PSBoundParameters              {}                                               
PSCommandPath                                                                   
PSCulture                      en-US                                            
PSDefaultParameterValues       {}                                               
PSEmailServer                                                                   
PSHOME                         D:\Windows\SysWOW64\WindowsPowerShell\v1.0       
PSScriptRoot                                                                    
PSSessionApplicationName       wsman                                            
PSSessionConfigurationName     http://schemas.microsoft.com/powershell/Micros...
PSSessionOption                System.Management.Automation.Remoting.PSSessio...
PSUICulture                    en-US                                            
PSVersionTable                 {PSVersion, WSManStackVersion, SerializationVe...
PWD                            D:\Windows\system32                              
req                            D:\local\Temp\Functions\Binding\6528a25c-61cc-...
REQ_HEADERS_ACCEPT             application/json, */*                            
REQ_HEADERS_ACCEPT-ENCODING    gzip, deflate, peerdist                          
REQ_HEADERS_ACCEPT-LANGUAGE    en-US, en-GB; q=0.8, en; q=0.6, nl-NL; q=0.4, ...
REQ_HEADERS_CONNECTION         Keep-Alive                                       
REQ_HEADERS_DISGUISED-HOST     stsfunctionappdemo.azurewebsites.net             
REQ_HEADERS_HOST               stsfunctionappdemo.azurewebsites.net             
REQ_HEADERS_MAX-FORWARDS       10                                               
REQ_HEADERS_ORIGIN             https://functions.azure.com                      
REQ_HEADERS_REFERER            https://functions.azure.com/?trustedAuthority=...
REQ_HEADERS_USER-AGENT         Mozilla/5.0 (Windows NT 10.0; Win64; x64) Appl...
REQ_HEADERS_WAS-DEFAULT-HOS... stsfunctionappdemo.azurewebsites.net             
REQ_HEADERS_X-ARR-LOG-ID       b61667f6-86d8-4bd1-a67c-bc0821ee2170             
REQ_HEADERS_X-ARR-SSL          2048|256|C=US, S=Washington, L=Redmond, O=Micr...
REQ_HEADERS_X-FORWARDED-FOR    217.122.212.62:4004                              
REQ_HEADERS_X-FUNCTIONS-KEY    EOm0H6fRai398YoJRGKkjzAZ7SV2E/zgnOjTaCOVs55W8h...
REQ_HEADERS_X-LIVEUPGRADE      1                                                
REQ_HEADERS_X-ORIGINAL-URL     /api/HttpTriggerPowerShellDemo?code=fYnNjQpSyo...
REQ_HEADERS_X-P2P-PEERDIST     Version=1.1                                      
REQ_HEADERS_X-P2P-PEERDISTEX   MinContentInformation=1.0, MaxContentInformati...
REQ_HEADERS_X-SITE-DEPLOYME... stsfunctionappdemo                               
REQ_METHOD                     GET                                              
REQ_QUERY_CODE                 fYnNjQpSyoUtXrR3fJ/vXn/L252RcTrzaeVFGP5vsMG6aa...
res                            D:\local\Temp\Functions\Binding\6528a25c-61cc-...
result                         {System.Management.Automation.PSVariable, Syst...
ShellId                        Microsoft.PowerShell                             
StackTrace                                                                      
true                           True                                             
VerbosePreference              SilentlyContinue                                 
WarningPreference              Continue                                         
WhatIfPreference               False
2017-01-29T16:01:52.976 Function completed (Success, Id=6528a25c-61cc-4d16-90ad-4869e47dc599)
2017-01-29T16:03:40  No new trace in the past 1 min(s).
[/sourcecode]
</pre>
</div>
<p>An interesting variable is “res” if we look at the value of this res variable we see that the value in my Azure Function is D:\local\Temp\Functions\Binding\677636fe-4384-42d2-94a4-55d14101ac99\res</p>
<p>But I was still wondering where this variable was configured because in the examples I often saw the variable being used to output the result.</p>
<p>Example where $res variable is being used:</p>
<div id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:f355e4b7-52de-4ac1-8c1e-b013424e7ffa" class="wlWriterEditableSmartContent" style="float: none;padding-bottom: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px"><pre>
[sourcecode language='powershell' ]
$Result = $psversiontable | ConvertTo-Json
Out-File -encoding Ascii -FilePath $res -inputObject $Result
[/sourcecode]
</pre>
</div>
<p><strong></strong> It turned out this variable is being configured in the integration configuration section of the Azure Function.</p>
<p><a href="https://stefanstranger.github.io/assets/image790.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;margin: 0px;padding-right: 0px" border="0" alt="image" src="https://stefanstranger.github.io/assets/image_thumb684.png" width="668" height="332"></a></p>
<p>So in above example the output result being show when the Azure Http Trigger PowerShell Function is being called stores the output of the Result variable in a file defined in the res variable.&nbsp; Hope this clarifies the $res variable seen in many examples on the internet about Azure PowerShell functions.</p>
<p>If you are also interested in the available environment variables within Azure PowerShell functions you can use the following PowerShell script in your Azure Function:</p>
<div id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:35c5511a-58ed-4adc-885e-20b4afe8cc01" class="wlWriterEditableSmartContent" style="float: none;padding-bottom: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px"><pre>
[sourcecode language='powershell' ]
write-Output &#039;Getting environment variables&#039;
$result = ls env: | out-string
write-output $result
[/sourcecode]
</pre>
</div>
<p><a href="https://stefanstranger.github.io/assets/image792.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;margin: 0px;padding-right: 0px" border="0" alt="image" src="https://stefanstranger.github.io/assets/image_thumb685.png" width="718" height="572"></a><br></p>
<p>I hope you have learned some new things about Azure PowerShell functions and you are interested to get started.</p>
<p>&nbsp;</p>
<p><strong>References:</strong></p>
<ul>
<li><a href="https://github.com/Azure/Azure-Functions/" target="_blank">Azure Function on Github</a> 
<li><a href="https://functions.azure.com/" target="_blank">Azure Functions Get Started</a> 
<li><a href="https://david-obrien.net/2016/07/azure-functions-PowerShell/" target="_blank">David O’Brien Azure Functions – PowerShell</a></li></ul>
