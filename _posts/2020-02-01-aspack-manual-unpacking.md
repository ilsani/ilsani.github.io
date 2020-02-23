---
title: ASPack manual unpacking
date:  2020-02-01 12:00:00
excerpt: "This post attempts to describe the approach to manual unpack a sample program packed using the ASPack packer."
---

This post attempts to describe the approach to manual unpack a program packed using the ASPack packer.

ASPack is an EXE packer created to compress Win32 executable files and to protect them against non-professional reverse engineering. Online can be found multiple resources descripting how to manually unpack an *aspacked* file. This post describe how a packed program has been unpacked using the WinDbg tool.

A free trial version of the ASPack packer can be downloaded at the official web site. For example, the 32 bit version can be found on the following URL: http://www.aspack.com/asprotect32.html.
However, a packed file has been found on Internet. Following an overview of the target file:


**Filename:** keygenme.exe<br/>
**MD5:** e2a1ba2227ba1dc32ac87449c0418df5<br/>
**File type:** PE32 executable (GUI) Intel 80386, for MS Windows<br/>
**File size:** 12 kb<br/>
**File:** ![Download]({{ site.url }}/assets/posts/aspack-manual-unpacking/keygenme.exe.zip)


An ASPacked file can be easly detected because has the `.aspack` section. Below are shown the PE sections of the target file and his `.aspack` section:

![peid-sections]({{ site.url }}/assets/images/posts/aspack-manual-unpacking/aspack-manual-unpacking-peid-sections.png)

Below is shown that the entry point of the executable is `00006001`, which is located on the `.aspack` section:

![peid-entrypoint]({{ site.url }}/assets/images/posts/aspack-manual-unpacking/aspack-manual-unpacking-peid-entrypoint.png)


In a nutshell, the following approach has been followed in order to unpack the packed file:
1. Find the first `pushad`
2. Put a breakpoint on the `esp` address after the push (1)
3. Hit the breakpoint (2)
4. Trace the code and follow the `retn` instruction
5. Dump and fix the unpacked file


First of all, it was calculated the entry point of the packed file.
This entry point has nothing to do with the original entry point of the unpacked file.

The entry point it was calculated using the `WinDbg` tool.

The `!peb` command displays the debugged Process Environment Block. Below is shown the `!peb` output. Specifically, the `ImageBaseAddress` contains the image base memory address.

![windbg-imagebaseaddress]({{ site.url }}/assets/images/posts/aspack-manual-unpacking/aspack-manual-unpacking-windbg-imagebaseaddress.png)

Since the `ImageBaseAddress` (`00400000`) has been found, the second part of the process entry point address should be calculated.
The `!dh` command can be used to found the `AddressOfEntryPoint`, which is defined as *the address of the entry point relative to the image base when the executable file is loaded into memory. For program images, this is the starting address*.

![windbg-addressofentrypoint]({{ site.url }}/assets/images/posts/aspack-manual-unpacking/aspack-manual-unpacking-windbg-addressofentrypoint.png)

Using the `ImageBaseAddress` and the `AddressOfEntryPoint` it was calculated the entry point of the packed file:
```
ImageBaseAddress + AddressOfEntryPoint = Entry Point
00400000         + 6001                = 00406001
```

During the analysis a breakpoint on the entry point has been created using the following command in WinDbg:
```
> bp 00400000+6001
```

It was noted that when the breakpoint on the entry point triggered, the first instruction is a `pushad`. Hence, the contents of the general-purpose registers were pushed onto the stack.

It is reasonabily to think that after the unpacking procedure the contents of these registers will be restored and the memory address on the stack, pointed to the `esp` register, will be accessed.

During the analysis it was created a breakpoint on the memory address pointed by the `esp` register. Below is shown the WinDbg command `ba r 4 12ffa4` used to create the breakpoint and the `esp` register which points to the `12ffa4` address.

![windbg-stack-breakpoint]({{ site.url }}/assets/images/posts/aspack-manual-unpacking/aspack-manual-unpacking-windbg-stack-breakpoint.png)

When the above breakpoint has been triggered, the unpacking routine decoded the file. Therefore, following the first `ret` instruction it was possible to found the original entry point.

Below is shown the code showing the `ret` instruction before the jump to the original entry point.

![windbg-ret]({{ site.url }}/assets/images/posts/aspack-manual-unpacking/aspack-manual-unpacking-windbg-ret.png)

It was noted that, after the above `ret`, the original entry point is on the `00401000` memory address. Below are shown the `Base Address` and the `End Address` on the WinDbg tool, respectively pointing to the `004010000` and `00407000` addresses.

![windbg-original-entry-point]({{ site.url }}/assets/images/posts/aspack-manual-unpacking/aspack-manual-unpacking-windbg-original-entry-point.png)

The `Import Reconstructor` tool contains a built-in process dumping tool. Therefore, it was used to dump the unpacked file on the file-system. As shown on the figure below, clicking on the middle window and selecting the `Advanced Command -> Select code section(s)` it was possible to `Full dump` the running unpacked code:

![dump]({{ site.url }}/assets/images/posts/aspack-manual-unpacking/aspack-manual-unpacking-dump.png)

Moreover, the `Import Reconstructor` tool has been also used to fix the IAT table. Therefore, using the original entyr point found on WinDbg (`004010000`), it was possible to use the `IAT Autosearch` Import Reconstructor button.
This locates the import address table from the original executable and allowed to create an executable unapacked file.

![iat-fix]({{ site.url }}/assets/images/posts/aspack-manual-unpacking/aspack-manual-unpacking-iat-fix.png)

As shown below it was possible to load the unpacked file on the IDA tool and retreiving all the used Win32 API:

![ida]({{ site.url }}/assets/images/posts/aspack-manual-unpacking/aspack-manual-unpacking-ida.png)



