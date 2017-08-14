---
layout: default
title:  "MS14-059 - Analysis and POC"
date:   2017-08-12 19:25:00
categories: aspnetmvc
---

{{ page.title }}
================

A few days ago I was on DotNetNuke's website and decided to look at the latest security updates. In this list, among other things, I saw that DNN has added support for a version of ASP.NET MVC released in 2016.

**Issue Title**<br />
*2017-06 (Low) Vulnerable ASP.NET MVC library (assembly) in Platform 8.0.0 and Evoq 8.3.0*

**Issue Summary**<br />
*Microsoft released a MVC vulnerability fix (KB2990942) a while ago. As DNN is using the MVC assembly from Microsoft, there is a need to update this assembly in DNN sites.*

**Mitigating factors**<br />
*The vulnerability could allow security feature bypass if an attacker convinces a user to click a specially crafted link or to visit a webpage that contains specially crafted content designed to exploit the vulnerability. <br />
By default, DNN distributions don't have any code utilizing the code that causes this vulnerability. But if you have a third party MVC module(s) you might be vulnerable.* <br />

Depending on the description in this advisory, it may be a Cross-Site Scripting (XSS) vulnerability but details are not included in the alert. So I decided to gather more information.

### Step #1 - Download the vulnerability fix (KB2990942)

In order to understand the applied fixes, the first step is to download the Microsoft patch, KB2990942.
KB2990942 is a msi archive that contains multiple files:
* System.Web.Mvc.dll
* System.Net.Http.dll
* System.Net.Http.Formatting.dll
* System.Net.Http.WebRequest.dll
* System.Web.Http.dll
* System.Web.Http.SelfHost.dll
* System.Web.Http.WebHost.dll

I decided to analyze the **System.Web.Mvc.dll**, that into this update is the 4.0.40804.0 version.

![Unzip KB2990942 msi]({{ site.url }}/assets/images/posts/ms14-059-analysis-and-poc/unzip-msi-fix.png)

### Step #2 - Download the unpatched DLLs

Once that I got the patched DLLs, I downloaded the previous version of the System.Web.Mvc dll from NuGet website. This task was easy because:
1. www.nuget.org lists all versions of the available Microsoft ASP.NET MVC DLLs
2. all these DLLs can be downloaded with the help of nuget.exe tool

According to the NuGet website the previous version of ASP.NET MVC 4.0.40804 is 4.0.30506.

![nuget download]({{ site.url }}/assets/images/posts/ms14-059-analysis-and-poc/nuget-download.png)

### Step #3 - DllDiff

### Step #4 - Exploit

## References
* [DNN Security Center](http://www.dnnsoftware.com/community/security/security-center)
* [Microsoft Security Fix - KB2990942](https://www.microsoft.com/en-us/download/details.aspx?id=44533)
* [Microsoft Security Bulletin MS14-059](https://technet.microsoft.com/en-us/library/security/ms14-059.aspx)