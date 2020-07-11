---
layout: post
title: PowerShell variables level 400
categories: [Scripting, PowerShell]
tags: [Scripting, PowerShell]
comments: true
---

**Contents**
- [Introduction](#introduction)
- [Get-Help About_Variables](#get-help-about_variables)
  - [VARIABLE NAMES THAT INCLUDE SPECIAL CHARACTERS](#variable-names-that-include-special-characters)
- [Environment Provider](#environment-provider)
- [References](#references)

# Introduction

While being a Microsoft Premier Field Engineer some years ago I regularly delivered different PowerShell workshops to customers and I always discussed PowerShell variables. So I thought I knew the ins and outs of PowerShell variables, but recently I saw something new about PowerShell variables which I was not aware of. It was in the blog post from <a href="https://twitter.com/jwtruher" target="_blank">James/Jim Truher</a> about "<a href="https://devblogs.microsoft.com/powershell/native-commands-in-powershell-a-new-approach/" target="_blank">Native Commands in PowerShell – A New Approach</a>"

In that blog post he used the following example:

```PowerShell
# retrieve data from REST endpoint
$baseUrl = "http://127.0.0.1:8001"
$urlPathBase = "api/v1/namespaces/default"
$urlResourceName = "pods"
$url = "${baseUrl}/${urlPathBase}/${urlResourceName}"
$data = (invoke-webrequest ${url}).Content | ConvertFrom-Json
```

And my interest was drawn to the part where the url variable was concatenated.

```PowerShell
$url = "${baseUrl}/${urlPathBase}/${urlResourceName}"
```

And then I also saw these Tweets in my timeline.

![Tweet](/assets/07-11-2020-1.png)

Time to investigate PowerShell variables in more depth.

# Get-Help About_Variables

So what did I teach during my PowerShell classes? Right, you start at the PowerShell help. 

```PowerShell
Get-Help About_Variables
```

A variable is a unit of memory in which values are stored. In Windows
PowerShell, variables are represented by text strings that begin with a
dollar sign ($), such as $a, $process, or $my_var.

Variable names are not case-sensitive. Variable names can include spaces
and special characters, but these are difficult to use and should be
avoided.

Ok I already knew that, but does the help file explains ${foo} = 'bar'?

Yes there is information in the About_Variables help file. Told you so to start with the Get-Help cmdlet ;-)

## VARIABLE NAMES THAT INCLUDE SPECIAL CHARACTERS

Variable names begin with a dollar sign. They can include alphanumeric
characters and special characters. The length of the variable name is
limited only by available memory.

Whenever possible, variable names should include only alphanumeric
characters and the underscore character (_).Variable names that include
spaces and other special characters, are difficult to use and should be
avoided.

To create or display a variable name that includes spaces or special
characters, enclose the variable name in braces. This directs PowerShell to
interpret the characters in the variable name literally.

For example, the following command creates and then displays a variable
named "save-items".

```PowerShell
    C:\PS> ${save-items} = "a", "b", "c"
    C:\PS> ${save-items}
    a
    b
    c
```

The following command gets the child items in the directory that is
represented by the "ProgramFiles(x86)" environment variable.

```PowerShell

    C:\PS> Get-childitem ${env:ProgramFiles(x86)}
```

To refer to a variable name that includes braces, enclose the variable name
in braces, and use the backtick (escape) character to escape the braces.
For example, to create a variable named "this{value}is" with a value of 1,
type:

```PowerShell
    C:\PS> ${this`{value`}is} = 1
    C:\PS> ${this`{value`}is}
    1
```

So now I've learned that you can enclose the variable name in braces if you want to use special characters in your variable name.

But this does not apply for the example code James Truher used in his blog post.

```PowerShell
# retrieve data from REST endpoint
$baseUrl = "http://127.0.0.1:8001"
$urlPathBase = "api/v1/namespaces/default"
$urlResourceName = "pods"
$url = "${baseUrl}/${urlPathBase}/${urlResourceName}"
$data = (invoke-webrequest ${url}).Content | ConvertFrom-Json
```
He creates 'normal' variable names without any special characters, like baseUrl, urlPathBase, urlResourceName and url, so why is he using 
```PowerShell
$url = "${baseUrl}/${urlPathBase}/${urlResourceName}"
```
to concatenate the 'normal' variable names into a new variable url?

# Environment Provider

And there is where Lee Holmes helps in his Tweet response.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Obscure fact - it&#39;s actually a shorthand for Get-Content and Set-Content, and works with any provider: <a href="https://t.co/CydXeLDnjL">https://t.co/CydXeLDnjL</a> <a href="https://t.co/lyKuIVfnlD">pic.twitter.com/lyKuIVfnlD</a></p>&mdash; Lee Holmes (@Lee_Holmes) <a href="https://twitter.com/Lee_Holmes/status/1280489154665566208?ref_src=twsrc%5Etfw">July 7, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

So Jim is here using Get-Content via the Variable Provider to the different variables. He is doing the following:

(Get-Content Variable:baseUrl) by using ${baseUrl}

Let's try this too:

```PowerShell
#Create a variable Foo with value Bar
$Foo = 'Bar'

#Retrieve variable using the Variable Provider
Get-Content Variable:Foo
```

![PowerShell code](/assets/07-11-2020-2.png)

I've learned something new about PowerShell variables, hope you also found this useful.

# References

* <a href="https://devblogs.microsoft.com/powershell/native-commands-in-powershell-a-new-approach/" target="_blank">Native Commands in PowerShell – A New Approach</a>
* <a href="https://mobile.twitter.com/sstranger/status/1280800338098954241" target="_blank">Access Environment Variables - Blog Post from Lee Holmes</a>