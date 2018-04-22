---
title: python网络爬虫和信息提取(1)
date: 2017-12-15
tags: [python, 网络爬虫]
---

## requests 库的安装和使用

### 安装

    pip install requests

### 7个主要方法

- requests.requests() 构造一个请求，支撑以下各方法的基础方法。
- requests.get() 获取HTML网页的主要方法，对应于HTTP的GET
- requests.head() 获取HTML网页头信息的方法，对应于HTTP的HEAD
- requests.post() 向HTML网页提交POST请求的方法，对应于HTTP的POST
- requests.put() 向HTML网页提交PUT请求的方法，对应于HTTP的PUT
- requests.patch() 向网页提交局部修改请求，对应于HTTP的PATCH
- requests.delete() 向HTML网页提交删除请求，对应HTTP的DELETE

其中`requests.requests()`方法是基础方法，其他方法都可以说是对他的封装。可以这样理解 `requests` 库只有一个方法，就是 `requests.requests()` 方法。

<!--more-->

#### requests()方法

`requests.requests()`方法是基础方法，其他方法都可以说是对他的封装。

```python
requests.requests(method, url, **kwargs)
```

**method: 请求方式，对应get/post/put等7种**

- `r = requests.requests('POST', url, **kwargs)`
- `r = requests.requests('GET', url, **kwargs)`
- `r = requests.requests('PUT', url, **kwargs)`
- `r = requests.requests('HEAD', url, **kwargs)`
- `r = requests.requests('PATCH', url, **kwargs)`
- `r = requests.requests('DELETE', url, **kwargs)`
- `r = requests.requests('OPTIONS', url, **kwargs)`

与HTTP协议对于的请求功能。前6种分别对应`requests.post()`、`requests.get()`等

**url: 拟获取页面的url链接**
**\*\*kwargs: 控制访问的参数，共13个。均为可选项**

- params: 字典或字节序列，作为参数增加到url中。

```python
kv = {'key1': 'value1', 'key2': 'value2'}
r = requests.requests('GET', 'http://python123.io/ws', params=kv)
print r.url  # http://python123.io/ws?key1=value1&key2=value2
```

- data：字典、字节序列或文件对象，作为Reques内容

```python
kv = {'key1': 'value1', 'key2': 'value2'}
r = requests.requests('POST', 'http://python123.io/ws', data=kv)
```

- json：JSON格式的数据，作为Request内容

```python
kv = {'key1': 'value1', 'key2': 'value2'}
r = requests.requests('POST', 'http://python123.io/ws', json=kv)
```

- headers：字典，HTTP定制头

```python
hd = {'user-agent': 'Chrome/10'}
r = requests.requests('POST', 'http://python123.io/ws', headers=hd)
```

- cookies：字典或CookieJar，Request中的cookie
- auth：元组，支持HTTP认证功能
- files：字典类型，传输文件

```python
fs = {'file': open('data.xls', 'rb')}
r = requests.requests('POST', 'http://python123.io/ws', files=fs)
```  

- timeout：设定超时时间，秒为单位

```python
r = requests.requests('GET', 'http://python123.io/ws', timeout=10)
```

- proxies：字典类型，设定访问代理服务器，可以增加登录认证

```python
pxs = {'http': 'http://user:pass@10.10.10.1:1234', 'https': 'https://10.10.10.1:4321'}
r = requests.requests('GET', 'http://www.baidu.com', proxies=pxs)
```

- allow_redirects：True/False，默认为True，重定向开关
- stream：True/Fasle，默认为True，获取内容立即下载开关
- verify：True/False，默认为True，认证SSL证书开关
- cert：本地SSL证书路径


#### get()方法

最简单的：`r = requests.get(url)`,构造一个向服务器请求资源的Requests对象。返回一个包含服务器资源的Response对象，包含爬虫返回的内容。

get方法的完整用法：`requests.get(url, params=None, **kwargs)`。其中

- url: 拟获取页面的url链接
- params：url中的额外参数，字典或字节流格式，可选。
- \*\*kwargs：12个控制访问的参数

这里演示`requests.get()`，请求百度首页，返回200状态码：

```python
import requests
r = requests.get('http://www.baidu.com')
print r.status_code  # 200表示请求成功
type(r)  # <class 'requests.models.Response'>
```

爬取网页的通用代码框架

```python
import requests


def getHtmlText(url):
    try:
        r = requests.get(url, timeout=30)
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        return r.text
    except:
        return '产生异常'

if __name__ == '__main__':
    url = 'http://www.baidu.com'
    print getHtmlText(url)
```

#### post()方法

通过`requests.post(url, data=None, json=None, **kwargs)`方法向服务器提交数据。如果我们提交的数据形式是键值对（字典）的形式时，会被自动放在`form`中，如果提交的是字符串时，则被存在`data`字段下。

提交键值对：

```python
payload = {'key1': 'value1', 'key2': 'value2'}
r = requests.post('http://httpbin.org/post', data=payload)
print r.text
''' {
  ...
  'form': {
    'key1': 'value1',
    'key2': 'value2'
  },
  ...
}
'''
```

提交字段：

```python
playload = '123456'
r = requests.post('http://httpbin.org/post', data=playload)
print r.text
'''
{
 'args': {},
 'data': '123456',
 'files': {},
 'form': {},
 'headers': {
   'Accept': '*/*',
   'Accept-Encoding': 'gzip, deflate',
   'Connection': 'close',
   'Content-Length': '6',
   'Host': 'httpbin.org',
   'User-Agent': 'python-requests/2.11.1'
 },
 'json': 123456,
 'origin': '183.199.3.79',
 'url': 'http://httpbin.org/post'
}
'''
```

#### head()方法

用`requests.head`获取一个链接，可以通过`r.headers`获取返回的头部信息内容，而 `r.text`发现是为空。通过该方法，可以用很少的网络流量获取网络资源的概要信息。
例如：
```python
import requests

r = requests.head('http://www.baidu.com')
print r.headers
# {'Content-Encoding': 'gzip', 'Server': 'bfe/1.0.8.18', 'Last-Modified': 'Mon, 13 Jun 2016 02:50:08 GMT', 'Connection': 'Keep-Alive', 'Pragma': 'no-cache', 'Cache-Control': 'private, no-cache, no-store, proxy-revalidate, no-transform', 'Date': 'Sat, 16 Dec 2017 03:26:25 GMT', 'Content-Type': 'text/html'}
print r.text  # ''
```

### Response对象的属性

|属性|说明|
| :------------------- | :----------------- |
|r.status_code|HTTP请求的返回状态，200表示连接成功，404表示失败|
|r.text|HTTP响应内容的字符串形式，即url对应的页面内容|
|r.encoding|从HTTP header中猜测响应内容编码方式|
|r.apparent_encoding|从内容中分析出的响应内容编码方式（备选编码方式）|
|r.content|HTTP响应内容的二进制形式|
| **r.raise_for_status()** | **如果不是200,产生异常requests.HTTPError** |


### Requests库异常

|异常|说明|
| :------------------- | :----------------- |
|requests.ConnectionError|网络连接错误异常，如DNS查询失败、拒绝连接等|
|requests.HTTPError|HTTP错误异常|
|requests.URLRequired|URL缺失异常|
|requests.TooManyReqirects|超过最大重定向次数，产生重定向异常|
|requests.ConnectTimeout|连接远程服务器超时异常|
|requests.Timeout|请求URL超时，产生超时异常|

---

### requests爬虫实战

这里通过以上的学习，掩饰几组使用requests库网络爬虫实战：

#### 图片爬取

```python
import requests
import os

url = "http://www.sinaimg.cn/dy/slidenews/1_img/2017_29/88490_1334396_633374.jpg"
root = "./images/"
path = root + url.split("/")[-1]

try:
    if not os.path.exists(root):
        os.makdir(root)
    if not os.path.exists(path):
        r = requests.get(url)
        whith open(path, "wb") as f:
            f.write(r.content)
        print "文件保存成功！"
    else:
        print "爬取失败！"
except:
    print "爬取失败"
```

#### ip地址判断

```python
import requests

url = "http://m.ip138.com/ip.asp?ip="
try:
    r = requests.get(url + "202.204.80.112")
    r.raise_for_status()
    r.encoding = r.apparent_encoding
    print r.text[-500:]
except:
    print "爬取失败！"
```
---
