---
title:  "CVE-2017-6078 - Crash in FastStone MaxView 3.1"
date:   2017-08-12 19:25:00
preview: "A crash in a BMP parser."
---

* _Author_: Martino Sani 
* _Release date_: 2017-02-20
* _Software_: http://www.faststone.org/FSMaxViewDownload.htm
* _Version_: 3.1 (Latest at the moment), 3.0 and probably previous versions
* _Vendor homepage_: http://www.faststone.org/
* _CVE_: CVE-2017-6078

FastStone MaxView is an image viewer that supports all major graphic formats.
Among the formats that it supports BMP is obviously present.

I decided to set up my Peach Fuzzer instance and fuzzing the FSMaxView's BMP parser.
BMP is a fairly simple format, I did not expect to find something... but... after a week of fuzzing... Peach claims to have found a fault.

[Here][1] you can download the (zipped) python script to reproduce the crash.

In a nutshell a BMP file starts with the following sections:

```
BITMAPFILEHEADER (14 bytes) + BITMAPINFOHEADER (40 bytes) + ...
```

BITMAPINFOHEADER has multiple fields that tell the detailed information about the image, which will be used to display the image on the screen. As one of these fields, *biSize* field specifies the size of the BITMAPINFOHEADER header. If *biSize* has a crafted value that not respect the real size of the BITMAPINFOHEADER header in the image, FSMaxView's BMP parser will crash with an `ACCESS_VIOLATION_EXCEPTION`.

WinDbg session:

```
# Open created POC file (poc.bmp) with FastStone MaxView
# Details
# 
# (1fc4.1d68): Access violation - code c0000005 (first chance)
#
# *** WARNING: Unable to verify checksum for image00000000`00400000
# *** ERROR: Module load completed but symbols could not be loaded for image00000000`00400000
#
# image00000000_00400000+0x9105:
# 00409105 2348fc          and     ecx,dword ptr [eax-4] ds:002b:fffffffc=????????
#
# 0:000:x86> r
# eax=00000000 ebx=0077a0f4 ecx=fffffff8 edx=f809bab0 esi=00000004 edi=fb50e9e0
# eip=00409105 esp=0019f514 ebp=0019f554 iopl=0         nv up ei pl nz na po nc
# cs=0023  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00210202
#
# 0:000:x86> kb
# ChildEBP RetAddr  Args to Child              
# WARNING: Stack unwind information not available. Following frames may be wrong.
# 00 0019f554 00621699 0002ff80 00000000 7fba0100 image00000000_00400000+0x9105
# 01 0019f5d4 005f4f2c 00000000 0019f640 00000000 image00000000_00400000+0x221699
# 02 0019f64c 005f3460 0019f6ac 005f3691 0019f6a4 image00000000_00400000+0x1f4f2c
# 03 0019f6a4 007470e1 0019f6b8 007470eb 0019f774 image00000000_00400000+0x1f3460
# 04 0019f774 0073e056 00000000 00000001 00000000 image00000000_00400000+0x3470e1
# 05 0019f7d4 006ab65f 0019f7e8 006ab73f 0019f828 image00000000_00400000+0x33e056
# 06 0019f828 0046ffbd fb48e1e0 00470be1 fb48e1e0 image00000000_00400000+0x2ab65f
# 07 0019f964 00456f70 fcf41418 77abf8f9 79d79524 image00000000_00400000+0x6ffbd
# 08 0019f9ac 00456beb 0019f9c0 00456c03 0019f9dc image00000000_00400000+0x56f70
# 09 0019f9dc 0042882a 0000b000 00000000 00000000 image00000000_00400000+0x56beb
# 0a 0019f9f4 777184f3 00090490 0000b000 00000000 image00000000_00400000+0x2882a
#
# .load C:\bin\peach-3.1.124-win-x64-debug\Debuggers\DebugEngine\msec64.dll
# !exploitable -m
#
# DESCRIPTION:Data from Faulting Address controls subsequent Write Address
# SHORT_DESCRIPTION:TaintedDataControlsWriteAddress
# CLASSIFICATION:PROBABLY_EXPLOITABLE
# EXPLANATION:The data from the faulting address is later used as the target for a later write.
#
#============================================================================================
```

## Timeline ##

* 2017-02-11: Vendor notification (1st attempt).
* 2017-02-15: Vendor notification (2nd attempt).
* 2017-02-20: No responses from the vendor. Disclosure.

The author is not responsible for the misuse of the information provided in this advisory.



[1]:{{ site.url }}/assets/data/cve2017-6078-fastone-maxview/crash.py.zip