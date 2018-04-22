---
title: 实现python tornado框架下的大文件秒传等技术（1）
date: 2017-07-27
tags: [tornado, python, 技术]
---


能不能实现这个功能，先做下去再说。


## 为什么选用tornado

Tornado 是一个Python web框架和异步网络库，主要是想使用它的异步非阻塞的性质，在前期可能体现不出tornado的特性，主要参考[这篇文章](https://www.idaima.com/article/17099)实现基本的功能，然后在深入tornado，在项目中充分发挥它的独到之处。这应该是一篇持久的更新博客，后期需要有tornado的开发经验。

## 搭建基本框架

之前一直在做django的项目，习惯也喜欢django的那套文件结构，在这个项目里，将仍然沿用这套结构。

<!--more-->

### 文件目录

如下图：

![Image 1.png](https://i.loli.net/2017/07/27/5979a07a6f3c2.png)

#### manage.py 入口文件，

启动项目时，`python manage.py`


```python
# coding: utf-8
import tornado.ioloop
import os
import sys

from tornado.options import options, parse_command_line
from urls import application

reload(sys)
sys.setdefaultencoding('utf8')


if __name__ == "__main__":
    parse_command_line()
    application.listen(options.port)
    tornado.ioloop.IOLoop.current().start()
```

#### settings.py 配置文件

- 端口：8888
- 静态文件目录：static
- 模板文件目录：templates


```python
# coding: utf-8
import os

from tornado.options import define, options, parse_command_line

BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))

define("port", default=8888, help="run on the given port", type=int)
define("debug", default=True, help="run in debug mode")
define("address", default="127.0.0.1", help="run in the given address")

settings = {
    "cookie_secret": "61oETzKXQAGaYdkL5gEmGeJJFuYh7EQnp2XdTP1o/Vo=",
    "template_path": os.path.join(os.path.dirname(__file__), "templates"),
    "static_path": os.path.join(os.path.dirname(__file__), "static"),
    "xsrf_cookies": False,
}
```

#### url.py 路由

配置 `/` 路由，跳转文件上传界面


```python
# coding: utf-8

import tornado.web

from settings import settings
from views import MainHandler

application = tornado.web.Application([
    (r"^/", MainHandler),

], **settings)
```


#### views.py 处理函数（视图）

tornado处理请求时不需要加上 return


```python
# coding: utf-8

import tornado.web
import os

class MainHandler(tornado.web.RequestHandler):

    def get(self):
        self.render("upload.html")
```


#### static 静态文件目录

这里存放 js、css、images等等静态文件，上传的文件也放在这。

#### templates 模板文件目录

这是放着所有的网页


[源代码在这里](https://github.com/suAdminWen/file_upload)
