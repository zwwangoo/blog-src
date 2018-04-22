---
title: 用 Tornado 实现 API 的初次尝试
date: 2018-03-17
tags: [python, tornado]
---

首先解释一下什么叫 API ：

> 应用程序接口（英语：Application Programming Interface，简称：API ），又称为应用编程接口，就是软件系统不同组成部分衔接的约定。由于近年来软件的规模日益庞大，常常需要把复杂的系统划分成小的组成部分，编程接口的设计十分重要。程序设计的实践中，编程接口的设计首先要使软件系统的职责得到合理划分。良好的接口设计可以降低系统各部分的相互依赖，提高组成单元的内聚性，降低组成单元间的耦合程度，从而提高系统的维护性和扩展性。(来自维基百科)

API 在多端中使用的更节省资源。网络应用程序，分为前端和后端两个部分。当前的发展趋势，就是前端设备层出不穷（手机、平板、桌面电脑、其他专用设备......）。因此，必须有一种统一的机制，方便不同的前端设备与后端进行通信。这导致API构架的流行。

关于RESTful的相关知识，参考[设计一套良好 REST API](https://zhuanlan.zhihu.com/p/34289466?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)

<!-- more -->

## 为什么选择 Tornado ?

在 Python 的众多WEB框架中，Tornado 它是异步非阻塞服务器，而且速度特别快。Tornado 每秒可以处理数以千计的连接，这意味着对于实时 Web 服务来说，Tornado 是一个理想的 Web 框架。

## 最基础的尝试

主要实现的是一个`ArticleHandler`，其中包含`GET`、`POST`、 `PUT`、 `DELETE`，实现基本的四种请求方式。当然，这是最简单的API设计的实现。

URL都是： /api/article[/id] POST请求不需要携带id，GET请求也可以不携带，但是意义不同

### API设计

这里的API设计是不太合理的，可以说URL的表现基本合理，但是 **数据的表现形式是不太合理的**，等待下一步进行优化处理。

- `GET`请求：`/api/article`，则返回一个列表 `[{...}, {...}, ...]`；
- `GET`请求：`/api/article/id`,如果 id 有效，且查找的到数据，则返回 `{...}` ；否则返回 `{}`；
- `POST`请求：`/api/article`, 需要携带参数，json 形式的 data 进行创建数据，如果参数错误或创建失败，返回 `{"error": 1, "code": ...}` ;否则返回创建的数据 `{...}`；
- `DELETE`请求： `/api/article/id`，如果id有效，且删除数据，并返回删除的数据 `{...}`；否则返回 `{"error": 1, "code": ...}`；
- `PUT`请求： `/api/article/id`，需要携带参数，json形式的data进行修改数据，需要数据的哪项值就可以传入哪项值，如果id有效且参数有效，则返回修改之后的数据 `{...}`，否则返回 `{}`；

### 详见代码

** `url`设计 **

```python
urls = [
    (r"/api/article/?(\w+)?", api.Article),
]
```

** `ApiHandler`实现 **

```python
# -*- coding: utf-8 -*-

import tornado.web
import datetime
import json

from settings import db
from eve.io.mongo import MongoJSONEncoder
from bson.objectid import ObjectId


class ApiHandler(tornado.web.RequestHandler):

    def data_received(self, chunk):
        pass

    def options(self):
        self.__set_response_header()

    # 这里的意思是在返回时附带允许请求的http response 头
    def __set_response_header(self):
        self.set_header('content-type', 'application/json')
        self.set_header("Access-Control-Allow-Origin", "*")
        self.set_header("Access-Control-Allow-Credentials", "true")
        self.set_header("Access-Control-Allow-Headers", "x-requested-with")
        self.set_header('Access-Control-Allow-Methods',
                        'POST, GET, OPTIONS, HEAD')


class Article(ApiHandler):
    """
    1、缺少基本认证机制
    2、没有验证tornado的异步的牛逼之处
    """
    @tornado.web.asynchronous
    def get(self, args=None):
        if args:
            try:
                article_list = db.article.find_one({"_id": ObjectId(args)})
                if not article_list:
                    raise Exception
            except Exception as err:
                print err
                article_list = {}
        else:
            article_list = list(db.article.find().sort('date', -1))
        self.finish(json.dumps(article_list, cls=MongoJSONEncoder, sort_keys=True))

    @tornado.web.asynchronous
    def put(self, args=None):
        article_id = args
        name = self.get_argument("name", None)
        content = self.get_argument("content", None)
        argument = {"name": name} if name else {}
        argument.update({"content": content}) if content else argument

        try:  # 这里对于id不合法的校验是不认真的，应该对传过来的参数进行校验
            if not content and not name:  # name 和 content 都不存在
                raise Exception
            db.article.update_one({"_id": ObjectId(article_id)},
                                  {'$set': argument}
                                  )
            article = db.article.find_one({"_id": ObjectId(article_id)})
        except Exception as err:
            print err
            article = {}
        return self.finish(json.dumps(article, cls=MongoJSONEncoder, sort_keys=True))

    @tornado.web.asynchronous
    def post(self, args=None):
        name = self.get_argument("name", None)
        content = self.get_argument("content", None)
        date = datetime.datetime.strftime(datetime.datetime.now(), "%Y-%m-%d %H:%M:%S")
        try:
            if name and content:
                article_id = db.article.insert_one({
                    "name": name,
                    "date": date,
                    "content": content
                }).inserted_id
                article = db.article.find_one({"_id": ObjectId(article_id)})
                return self.finish(json.dumps(article, cls=MongoJSONEncoder, sort_keys=True))
            else:
                self.finish(json.dumps({"error": 1, "code": "0002"}))
                return
        except Exception as err:
            print err
            self.finish(json.dumps({"error": 1, "code": "0001"}))
            return

    @tornado.web.asynchronous
    def delete(self, args=None, **kwargs):
        if not args:
            return self.finish(json.dumps({"error": 1, "code": "0003"}))
        try:
            article = db.article.find_one({"_id": ObjectId(args)})
            if article:
                db.article.delete_one({"_id": ObjectId(args)})
            else:
                article = {}
            return self.finish(json.dumps(article, cls=MongoJSONEncoder, sort_keys=True))
        except Exception as err:
            print err
            return self.finish(json.dumps({"error": 1, "code": "0001"}))

```

**其他tornado的配置自行补充，这里不说明配置啦**

## 总结

其实具体的API设计原则需要有各个项目组或者开发团队进行设计和统一规定，没有具体的标准之说，具体的取舍，在项目开发中就会体现处其中的好处。本文是首次接触API的设计原则问题，只是在这里记录自己现在的想法，但是 **不能保证以上所有的论述和实现方式都是合理的** ，**仅供自己的学习和参考**，等后期的深入，本人会继续更新最新的文章，更新其中不足的地方，再次共勉！

---


## 解决用户登录的问题

参考项目代码，重新整理了认证的部分，发现tornado自带的功能已经很强大啦，这不在这里补上认证的部分，其实原理很简单：

用户登录之后返回一个token的值，然后再以后的请求头部中需要携带该token的值，来看看代码：

```python

# api.py
class Login(ApiHandler):
    """
    这里是登录获取Token的API，当然这里的处理是如果用户不存在就创建token
    """
    def post(self):
        name = self.get_argument("name", None)
        args = {"name": name}
        user = db.users.find_one(args)
        if not user:
            user = {
                "name": name,
                "password": "123456",
                "sex": "男"
            }
            user_id = db.users.insert_one(user).inserted_id
            user.update({"_id": str(user_id)})
        else:
            user_id = user["_id"] = str(user["_id"])

        token = self.create_signed_value('tid', str(user_id), version=2)
        user.update({"token": token})

        return self.finish(rtjson(**user))


class ApiHandler(tornado.web.RequestHandler):

    def data_received(self, chunk):
        pass

    def get_current_user(self):
        return self.get_secure_cookie("tid", self.request.headers.get('token', None))

    def get_argument(self, name, default=None, strip=True):
        # 重载该方法
        if self.request.method != "GET":
            if 'application/json' in str(self.request.headers.get('Content-Type')) and self.request.body and self.request.body != '{}' \
                    and self.request.body.startswith('{'):
                obj = json.loads(self.request.body)
                return obj.get(name, default)
            elif 'application/x-www-form-urlencoded' in str(self.request.headers.get('Content-Type')) \
                    and self.request.body and self.request.body != '{}' \
                    and self.request.body.startswith('{'):
                obj = json.loads(self.request.body)
                return obj.get(name, default)
        return self._get_argument(name, default, self.request.arguments, strip)

    def options(self):
        self.__set_response_header()

    # 这里的意思是在返回时附带允许请求的http response 头
    def __set_response_header(self):
        self.set_header('content-type', 'application/json')
        self.set_header("Access-Control-Allow-Origin", "*")
        self.set_header("Access-Control-Allow-Credentials", "true")
        self.set_header("Access-Control-Allow-Headers", "x-requested-with")
        self.set_header('Access-Control-Allow-Methods',
                        'POST, GET, OPTIONS, HEAD')


# base.py
# 这是封装的一个返回合法json的方法

import json
import time
from eve.io.mongo import MongoJSONEncoder


def rtjson(code=1, **args):
    if code == 1:
        args['status'] = 1
        args['response_time'] = int(time.time())
    else:
        args['status'] = 0
        args['error_code'] = code
        # args['error_msg'] = errorDesc.get(code)
        args['response_time'] = int(time.time())

    return json.loads(json.dumps(args, cls=MongoJSONEncoder, sort_keys=True))

```


## 再次总结

这里虽然实现了登录授权的问题，但是没有实现的是分级的问题，即不同用户的权限不同的处理。后期再处理吧。目前所有的代码请参考[https://github.com/suAdminWen/restapi](https://github.com/suAdminWen/restapi)
