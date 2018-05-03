---
title: nginx+uwsgi实现负载均衡
date: 2017-05-30
tags: [Linux, nginx]
---

>nginx不单可以作为强大的web服务器，也可以作为一个反向代理服务器，而且nginx还可以按照调度规则实现动态、静态页面的分离，可以按照轮询、ip哈希、URL哈希、权重等多种方式对后端服务器做负载均衡，同时还支持后端服务器的健康检查。在本文中中，我们使用nginx和uwsgi的方式来实现负载均衡。


本文测试环境是VMware里安装了三台一样环境的ubuntu server服务器，内网ip分别为：A 192.168.102.133，B 192.168.102.134 C 192.168.102.135，此三台服务器部署相同的项目环境，使用同一个数据库，都已经安装uwsgi。A服务器作为主服务器，B、C为副服务器。数据库安装在A服务器上。即可以看成A服务器，能够单独提供完整的服务（如何部署一个完整的服务请参考[这里](https://suadminwen.github.io/nginx-+-uwsgi%E9%83%A8%E7%BD%B2odoo%E6%9C%8D%E5%8A%A1)）。

<!--more-->


## A服务器的配置

### 1 uwsgi的配置文件

```
<uwsgi>
    <socket>192.168.102.133:8077</socket>   <!-- 这里填写的是本机的ip -->
    <chdir>/home/odoo-10</chdir>
    <wsgi-file>wsgi-odoo.py</wsgi-file>
    <processes>8</processes> <!-- 进程数 -->
    <workers>4</workers>  <!--  -->
    <daemonize>uwsgi.log</daemonize>
    <py-autoreload>1</py-autoreload>
</uwsgi>

```

### 2 nginx的配置

在`http{}`中添加以下内容

```
upstream myodoo {
    server 192.168.102.133:8077;
    server 192.168.102.134:8077;
    server 192.168.102.135:8077;
}
```
在`server{}`中添加以下内容

```
location / {
  include        uwsgi_params;
  uwsgi_pass     myodoo;
  uwsgi_param UWSGI_SCRIPT home.odoo-10.wsgi-odoo;
  uwsgi_param UWSGI_CHDIR /home/odoo-10 ;
}
```

### 3 重启nginx和uwsgi


## B和C服务器的配置


B和C服务器不需要安装nginx，uwsgi的配置同A相同，唯一不同的地方就是

    192.168.102.134:8077

    192.168.102.133:8078

这里要填写自己的ip

添加完成之后，重启uwsgi


## 注意


- 三台服务器共用数据库和一套代码，在vm下，最好是将一台服务器配置完成之后，克隆成其他两台服务器就行。
- 负载均衡没有限制有几台服务器，本文只是演示三台服务器。
- 测试有没有成功，增加访问主服务器的进程数，然后查看uwsgi.log就可以看到负载均衡的效果。
- 祝你好运