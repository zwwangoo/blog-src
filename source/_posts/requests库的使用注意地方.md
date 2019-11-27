---
title: requests库的使用注意地方
date: 2019-10-06
tags: [python]
---

1、设置cookie

```
from requests import Session
from requests.cookies import RequestsCookieJar

session = Session()
res = session.post('http://data.huaxuejia.cn/search.php',
files={'search_keyword': (None, casnum)})

cookie_jar = RequestsCookieJar()
resd = requests.utils.dict_from_cookiejar(res.cookies)
cookie_jar.set([key for key in resd][0], resd[[key for key in resd][0]], domain='.huaxuejia.cn')

res = session.post('http://data.huaxuejia.cn/search.php', files={'search_keyword': (None, casnum)}, cookies=cookie_jar)

```

2、post提交表单

```
from requests import Session

res = session.post(url, files={key: (None, value)})
```



3、错误（too many open files）:

```
OpenSSL.SSL.Error: [('system library', 'fopen', 'Too many open files'), ('BIO routines', 'BIO_new_file', 'system lib'), ('x509 cetificate routines', 'X509_load_cert_crl_file', 'system lib')]
```

解决：

```
import requests

session = requests.Session()
res = session.get('http://www.baidu.com')
session.close()
del(session)
```
