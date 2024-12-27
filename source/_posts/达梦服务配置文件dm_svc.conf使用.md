---
title: 达梦服务配置文件dm_svc.conf使用
date: 2024-09-06
tags:
  - 数据库
---

dm_svc.conf是客户端连接的服务配置文件，在客户端(jdbc、dpi等)使用服务名进行数据库连接的情况下，需要对dm_svc.conf进行配置，如果已经安装了达梦数据库软件，会默认在安装的服务器目录上生成此配置文件，文件的位置分别位于

在windows平台：
- 64位的DM安装在Win64操作平台下，此文件位于%SystemRoot%\system32目录；
- 32位的DM安装在Win64操作平台下，此文件位于%SystemRoot%\SysWOW64目录；

在Linux平台下，此文件位于/etc目录。

对于未安装dm数据库的情况下，可以手工生成dm_svc.conf文件，此文件可以放在默认的文件路径：

windows:` %SystemRoot%\system32(SysWOW64)\dm_svc.conf `

linux: `/etc/dm_svc.conf `

也可以根据不同的场景放置于非默认目录，但是此时的jdbc连接串中需要配置参数dmsvcconf，指定此参数的具体路径。
当然，在配置文件中的参数也可以直接写入在jdbc等的连接字符串中，针对以上三种情况下的jdbc连接串示例如下：

1.     dm_svc.conf位于默认位置：jdbc:dm://dm_rww  --这里的dm_rww是文件中配置的服务名称。
2.     dm_svc.conf位于自定义位置：jdbc:dm://?dmsvcconf=/opt/dm_svc.conf
3.     无dm_svc.conf文件：jdbc:dm://dmconn?dmconn=(192.168.56.13:32141,192.168.56.14:32142,192.168.56.15:32143)&rw_Separate=(1)&rw_Percent(0)&login_mode=(1)  --其他参数可继续添加

此文件中各参数的配置格式：
参数名=(参数值)
此文件的常用参数(摘自达梦系统管理员手册，更多参数意义参考管理员手册)：

- https://www.cnblogs.com/ly-nye/p/16573648.html