---
title: nginx+uwsgi部署django服务
date: 2017-05-30
tags: [Linux, nginx, django]
---

这篇文章很早以前就自己整理出来了，当然也是参考网上的教程，但是当时也是费劲了千辛万苦，在此做一份记录。

## ubuntu下配置nginx + uwsgi 参考这篇文章[nginx+uwsgi部署odoo服务](https://suadminwen.github.io/2017/05/29/nginx%20+%20uwsgi%E9%83%A8%E7%BD%B2odoo%E6%9C%8D%E5%8A%A1/)



## 配置django：

编写django_wsgi.py文件，将其放在与文件manage.py同一个目录下。
注意：
编写文件时需要注意语句`os.environ.setdefault`比如，如果你的项目为myNote，则你的语句应该是

<!--more-->

    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "myNote.settings")

```
    #!/usr/bin/env python
    # coding: utf-8
    import os
    import sys
    # 将系统的编码设置为UTF8
    reload(sys)
    sys.setdefaultencoding('utf8')
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "myNote.settings")
    from django.core.wsgi import get_wsgi_application
    application = get_wsgi_application()
```

## 连接django和uwsgi，实现简单的WEB服务器：

假设你的Django项目的地址是/home/wen/myNote，可以执行以下命令：

```
uwsgi --http-socket :8000 --chdir /home/wen/myNote --plugin python --module django_wsgi
```

这是可以通过访问`127.0.0.1:8000`来查看服务是否正常。

为了实现Nginx与uWSGI的连接，两者之间将采用soket来通讯方式。在本节中，我们将使用uWSGI配置文件的方式来改进uWSGI的启动方式。假定你的程序目录是`/home/wen/myNote`我们将要让Nginx采用8077端口与uWSGI通讯，请确保此端口没有被其它程序采用。
新建一个XML文件：`djangochina_socket.xml`，将它放在 `/home/wen/myNote`
内容如下：
```
<uwsgi>
    <socket>:8077</socket>
    <chdir>/home/wen/myNote</chdir>
    <module>django_wsgi</module>
    <processes>4</processes> <!-- 进程数 --> 
    <daemonize>uwsgi.log</daemonize>
</uwsgi>
```

在上面的配置中，我们使用 `uwsgi.log` 来记录日志，开启4个进程来处理请求。这样，我们就配置好uWSGI了。

## 配置Nginx

我们假设你将会把Nginx程序日志放到你的目录下`/var/log/error.log`，请确保该目录存在。

我们假设你的Django的static目录是`/home/wen/myNote/webStatic/`， 
我们假设你的域名是 `127.0.0.1` （在调试时你可以设置成你的机器IP）
我们假设你的域名端口是 `8001`（在调试时你可以设置一些特殊端口如 8070）

基于上面的假设，我们为`/etc/nginx/nginx.conf`添加以下配置(命令locate nginx.conf可以找到conf文件的路径)
```
server {
    listen   8001;
    server_name 127.0.0.1;
    access_log /var/log/access.log;
    error_log /var/log/error.log;
    #charset koi8-r;
    #access_log  logs/host.access.log  main;
    location / {
     include        uwsgi_params;
     uwsgi_pass     127.0.0.1:8077;
    }
    #error_page  404              /404.html;
    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
    location /static/ {
    alias  /home/wen/myNote/webStatic/;
    index  index.html index.htm;
    }
}
```

## Nginx+uWSGI+Django的实现方式

在完成上面配置后，需要按以下步骤来做：

### 重启Nginx服务器，以使Nginx的配置生效。

    nginx -s  reload

### 重启后检查Nginx日志是否有异常。

启动uWSGI服务器

```
cd /home/wen/myNote/
uwsgi -x djangochina_socket.xml
```

检查日志 uwsgi.log 是否有异常发现。

### 访问服务

因此，我们访问 `127.0.0.1:8001`，如果发现程序与 单独使用Django启动的程序一模一样时，就说明成功啦！

**如果打不开网页，可能是端口被占用，或者防火墙的问题**

### 关闭服务的方法

将uWSGi进程杀死即可。

su进入root 

命令`killall -9 nginx`

命令`killall -9 uwsgi`