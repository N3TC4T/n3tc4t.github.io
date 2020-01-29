---
layout: post
title: Hide Django App From Wappalyzer Detection System
categories: posts
comments: true
en: true
description: Hide Django App From Wappalyzer Detection System
keywords: "django, wappalyzer, hide, detect, csrf"
---


# Introduction


__what is [Wappalyzer](https://wappalyzer.com/) ?__


_Identify software on the websites you visit_

_Wappalyzer is a browser extension that uncovers the technologies used on websites. It detects content management systems, eCommerce platforms, web servers, JavaScript frameworks, analytics tools and many more._


__Why we need to hide our django app from Wappalyzer?__

we see daily new exploits and vulnerabilities has discovered and if we don't patch them so the risk of hacking of our system will be increase .
when some 0day bugs has been released many hackers with their bots and scripts looking for vuln systems , and with hiding our system from those bots we can sure our system is hidden from their eyes .

__how wappalyzer detect our system is using django ?__

with a simple looking on wappalyzer sourcecode we understand they use "csrfmiddlewaretoken" field name to detect we using django :

```
"Django": {
        "cats": [
                    18
            ],
            "env": "^__admin_media_prefix__",
            "html": "(?:powered by <a[^>]+>Django ?([\\d.]+)?|<input[^>]*name=[\"']csrfmiddlewaretoken[\"'][^>]*>)\\;version:\\1",
            "icon": "Django.png",
            "implies": "Python",
            "website": "djangoproject.com"
    },
```


so with changing our csrf token field name we will bypass it .

# Getting start

for changing "csrfmiddlewaretoken" field name we should change two file in our django base lib directory :

* Template tag file  (`django/template/defaulttags.py`)
* CSRF middleware handler file (`django/middleware/csrf.py`)

_in new django version the template tag module is under this file : `django/template/backends/utils.py`_

for better hidding I use signing to create new CSRF token field name , so the field name will be different in every project (by secret key)

__changing csrf field name in Template tag__

_file:django/template/defaulttags.py_
_line:59_

```
from django.core import signing
```

```
csrf_field = signing.dumps('csrfmiddlewaretoken').partition(':')[0]
return format_html("<input type='hidden' name='{0}' value='{1}' />", csrf_field, csrf_token)
```

with this change this module will return something like this instead of our default "csrfmiddlewaretoken"  :

```
<input type='hidden' name='ImNzcmZtaWRkbGV3YXJldG9rZW4i' value='65jbHMBF3hZ7uTOU7a0dHEQp6zqtNgm4' />
```

__changing csrf field name in CSRF middleware__

_file:django/middleware/csrf.py_
_line:173_

```
from django.core import signing
```

```
request_csrf_token = ""
if request.method == "POST":
    csrf_field = signing.dumps('csrfmiddlewaretoken').partition(':')[0]
    request_csrf_token = request.POST.get(csrf_field, '')
```


# anything else?

until now we could hide our django system from wappalyzer .
wappalyzer is a script and we can bypass it but we cannot bypass hackers minds , so we should change everything that can show we using django our anything else ,like admin url etc ...


