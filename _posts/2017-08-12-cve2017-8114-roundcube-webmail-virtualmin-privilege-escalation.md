---
title:  "Roundcube Webmail Virtualmin Privilege Escalation (CVE-2017-8114)"
date:   2017-08-12 19:25:00
excerpt: "Roundcube Webmail allows arbitrary password resets by authenticated users. This affects versions before 1.0.11, 1.1.x before 1.1.9, and 1.2.x before 1.2.5. The problem is caused by an improperly restricted exec call in the virtualmin and sasl drivers of the password plugin."
---

I discovered this security issue (CVE-2017-8114) during my for fun and no-profit code review activities.

[Roundcube Webmail](https://roundcube.net/){:target="_blank"} is a browser-based multilingual IMAP client with an application-like user interface. Roundcube version 1.3-beta and probably previous versions do not properly sanitize client's inputs. This can lead to an arbitrary password resets by authenticated users.

Roundcube uses multiple plugins that extend its functionalities. They are not part of the core application but can be installed individually. Roundcube comes with a number of plugins, and more third-party plugins are available for download.

Official *Password* plugin in its *virtualmin* driver allows to an attacker, that has a valid username/password to login in his web panel, to execute malicious inputs. This could allow to an attacker to reset victim's password and in some scenarios getting a system shell.

## Technical Details ##

`Save()` method in `virtualmin.php` uses an `exec()` function without properly filter user-supplied inputs:
```php
class rcube_virtualmin_password {

   function save($currpass, $newpass) {

      ...

      $username = escapeshellcmd($username);
      $domain   = escapeshellcmd($domain);
      $newpass  = escapeshellcmd($newpass);
      $curdir   = RCUBE_PLUGINS_DIR . 'password/helpers';

      exec("$curdir/chgvirtualminpasswd modify-user --domain $domain --user $username --pass $newpass", $output, $returnvalue);

      ...

   }

}
```

`escapeshellcmd()` escapes any characters in a string that might be used to trick a shell command into executing arbitrary commands. This function should be used to make sure that any data coming from user input is escaped before this data is passed to the `exec()` or  `system()` functions, or to the backtick operator.

`escapeshellarg()` adds single quotes around a string and quotes/escapes any existing single quotes allowing you to pass a string directly to a shell function and having it be treated as a single safe argument. This function should be used to escape individual arguments to shell functions coming from user input. The shell functions include `exec()`, `system()` and the backtick operator.

Thanks to `escapeshellcmd()` an attacker can not execute arbitrary commands, but can use arbitrary `modify-user` parameters.

`chgvirtualminpasswd` is a simple wrapper to [virtualmin](https://www.virtualmin.com/) that must run as root, which makes this injection more interesting. For the list of `modify-user` parameters see [virtualmin-doc](https://www.virtualmin.com/documentation/developer/cli/modify_user).

As example, a malicious user can inject custom string as `$newpass` and reset the password of an arbitrary user.
Attacker, authenticated as `mark`, can insert following string as new password:
```
foo --user john --pass hacked 
```

`Exec()` will execute following command:
```
chgvirtualminpasswd modify-user --domain localhost --user mark --pass foo --user john --pass hacked
```

last `--user` and `--pass` parameters will be evaluated by `chgvirtualminpasswd` and the victim’s (john) password will be changed.

If ssh service is open and reachable, the attacker can login using the victim's credentials. Or, if ssh service is reachable, but users can't login via an interactive shell (e.g. have a /bin/nologin or /dev/null shell), the attacker can change his shell (e.g. to /bin/bash) injecting below string as `$newpass`:
```
pass --shell /bin/bash
```

Vendor promptly fixes the issue and a similar bug in `sasl` driver. [(Roundcube Project News)](https://roundcube.net/news/2017/04/28/security-updates-1.2.5-1.1.9-and-1.0.11)

## Timeline ##

* 2017-04-24: Vendor notification
* 2017-04-25: Vendor fixes the bug on dev branch
* 2017-04-27: Vendor releases a beta version
* 2017-04-28: Vendor releases a stable version

The author is not responsible for the misuse of the information provided in this advisory.