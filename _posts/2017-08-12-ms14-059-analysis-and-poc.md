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

I had two DLLs: patched and unpatched version of System.Web.Mvc.dll.

The simplest way to find the differences between patched and unpatched System.Web.Mvc.dll is compare this two DLLs. There are multiple methods to do this, but since System.Web.Mvc.dll has been wrote in C# Microsoft .NET language I used the following approch:

1. I used JetBrains dotPeek to decompile these 2 DLLs
2. I exported the source code files to the file system (Right-click on the DLL -> Export to Project)
3. With the help of WinMerge I compared these 2 projects in order to find the differences

*If you try to reproduce the step (3) remember to add the following Linefilters in WinMerge, otherwise you will get a lot of junk as result:*
```
^// MVID:
^// Assembly:
```

WinMerge found 41 modified files. A lot of these findings were false positives due to the compiler optimization or decompiler issues. Others were changes not related to a security fix. But 6 of these files were interesting:
* DefaultDisplayTemplates.cs
* DefaultEditorTemplates.cs
* CachedDataAnnotationsModelMetadata.cs
* CachedModelMetadata1.cs
* DataAnnotationsModelMetadataProvider.cs
* ModelMetadata.cs

The patched version of these files have code related to the HTML encoding. For example, *DefaultDisplayTemplates.cs* has a new *HtmlTemplate.Encode()* call in the *ObjectTemplate()* method.
```
internal static string ObjectTemplate(HtmlHelper html, TemplateHelpers.TemplateHelperDelegate templateHelper)
{
	...
	
	if (modelMetadata.Model == null)
           return modelMetadata.NullDisplayText;

	if (cDisplayClass5.templateInfo.TemplateDepth > 1)
	{
		string str = modelMetadata.SimpleDisplayText;

		// new code encodes HTML string
		if (modelMetadata.HtmlEncode)
		   str = html.Encode(str);
		return str;
	}

	...

}
```

After analysis all the changes reported by WinMerge looks like that MS14-059 fixes some Cross-Site Scripting (XSS) issues in the System.Web.Mvc.dll.

### Step #4 - Exploit



## References
* [DNN Security Center](http://www.dnnsoftware.com/community/security/security-center)
* [Microsoft Security Fix - KB2990942](https://www.microsoft.com/en-us/download/details.aspx?id=44533)
* [Microsoft Security Bulletin MS14-059](https://technet.microsoft.com/en-us/library/security/ms14-059.aspx)
* [HtmlHelper.Encode()](https://msdn.microsoft.com/en-us/library/system.web.mvc.htmlhelper.attributeencode(v=vs.118).aspx#M:System.Web.Mvc.HtmlHelper.AttributeEncode(System.String))