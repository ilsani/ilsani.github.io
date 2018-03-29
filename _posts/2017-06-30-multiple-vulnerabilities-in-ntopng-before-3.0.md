---
title: "Multiple vulnerabilities in ntopng before v. 3.0"
date: 2017-06-30 00:00:00
preview: "Multiple vulnerabilities have been identified in ntopng, the next generation version of the original ntop, a network traffic probe that monitors network usage."
---

* _Author_: Martino Sani
* _Version_: before 3.0
* _CVE_: CVE-2017-7416, CVE-2017-7458, CVE-2017-7459

ntopng is the next generation version of the original ntop, a network traffic probe that monitors network usage. ntopng is based on libpcap and it has been written in a portable way in order to virtually run on every Unix platform, MacOSX and on Windows as well.

Multiple issues have been identified in ntopng before version 3.

## CVE-2017-7416

ntopng before 3.0 allows multiple XSS because GET and POST parameters are improperly validated.

## CVE-2017-7458

The `NetworkInterface::getHost` function in `NetworkInterface.cpp` in ntopng before 3.0 allows remote attackers to cause a denial of service (NULL pointer dereference and application crash) via an empty field that should have contained a hostname or IP address. 

## CVE-2017-7459

ntopng before 3.0 allows HTTP Response Splitting attack.

## References
* [ntopng website](https://www.ntop.org)
* [CVE-2017-7416](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-7416)
* [CVE-2017-7458](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-7458)
* [CVE-2017-7459](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-7459)
