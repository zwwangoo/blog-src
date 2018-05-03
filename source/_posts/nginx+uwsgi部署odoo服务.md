---
title: nginx+uwsgi部署odoo服务
date: 2017-05-29
tags: [Linux, nginx]
---

> 说明：
> - 用nginx和uwsgi部署odoo的前提是odoo的项目本身服务跑起来没有错误，这里不再介绍odoo的环境搭建
> - 文章中，我的项目名为 odoo-10,项目绝对路径为/home/odoo-10
> - 注意ubuntu用户类型，一般是直接进入root用户，如果不是，请执行命令的时候全程带着sudo

## 安装配置uwsgi

### 下载安装uwsgi

     sudo apt-get install uwsgi uwsgi-plugin-python

<!--more-->

### 测试uwsgi安装成功

在你的机器上写一个test.py

```python
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return "Hello World"
```

然后执行shell命令：

    uwsgi --http :8001 --wsgi-file test.py

访问网页：

    http://127.0.0.1:8001/

看在网页上是否有Hello World


### 报错

#### 如果报chdir() nosuch file or directory[core/uwsgi.c line 2581]之类的错：

执行一下命令：

    sudo apt-get install build-essential python-dev

#### 报错：

```
uwsgi: option '--http' is ambiguous; possibilities: '--http-socket' '--http-socket-modifier2' '--http-socket-modifier1'
getopt_long() error
```

将命令改为

    uwsgi --http-socket :8001 --plugin python --wsgi-file test.py

原因：使用 uwsgi 时都会碰到uwsgi: unrecognized option '--uwsgi-file'如 --module , --wsgi-file , --callable 等，需要在上面那些未识别选项前加上 --plugin python 来告诉 uWSGI 我在使用 python 插件，后面那些选项你 用python 插件去解析。

### 编写odoo_wsgi.py，将其放在项目根目录下，比如这里，我放的路径就是/home/odoo-10/odoo_wsgi.py

```python
import odoo
odoo.multi_process = True
odoo.conf.server_wide_modules = ['web']
conf = odoo.tools.config
conf['addons_path'] = '/home/odoo-10/odoo/addons'  # 这里路径为odoo目录下addons的路径
conf['proxy_mode'] = 'True'
conf['debug_mode'] = 'False'
application = odoo.service.wsgi_server.application
odoo.service.server.load_server_wide_modules()
```

### 连接odoo和uwsgi，实现简单的WEB服务器

    uwsgi --http-socket :8001 --chdir /home/odoo-10 --plugin python --wsgi-file odoo_wsgi.py

访问网页：

    http://127.0.0.1:8001/


看看是否和单跑odoo时提供相同的服务，祝你好运

### 为了实现Nginx与uWSGI的连接，两者之间将采用soket来通讯方式。

本文我们将使用uWSGI配置文件的方式来改进uWSGI的启动方式。

我们将要让Nginx采用8077端口与uWSGI通讯，请确保此端口没有被其它程序采用。

新建一个XML文件：
odoochina_socket.xml，将它放在 odoo_wsgi.py同级目录下

内容如下：

```
<uwsgi>
    <socket>:8077</socket>
    <chdir>/home/odoo-10</chdir>
    <module>django_wsgi</module>
    <processes>4</processes> <!-- 进程数 -->
    <daemonize>uwsgi.log</daemonize>
</uwsgi>
```

在上面的配置中，我们使用 uwsgi.log 来记录日志，开启4个进程来处理请求。

这样，我们就配置好uWSGI了。

## 安装配置nginx

### 安装nginx

    sudo apt-get install nginx

### 配置nginx

打开nginx.conft添加如下内容到 http{
}的最后

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
      uwsgi_param UWSGI_SCRIPT home.odoo-10.wsgi-odoo;
      uwsgi_param UWSGI_CHDIR /home/odoo-10 ;
   }
 }
```

### 检查配置文件有没有错误

    nginx -t

结果如下：

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

则，nginx无语法错误，至于有没有其他的错误，那就要看运气了

## Nginx+uWSGI+odoo的实现方式

### 重启Nginx服务器，以使Nginx的配置生效

    nginx -s  reload

### 启动uWSGI服务器

    cd /home/odoo-10
    uwsgi --plugin python -x odoochina_socket.xml

### 访问服务

基于上面的假设你的域名是127.0.0.1

因此，我们访问 127.0.0.1:8001，如果发现程序与 单独使用odoo启动的程序一模一样时，就说明成功啦！

### 关闭服务

    killall -9 nginx
    killall -9 uwsgi
