---
title: mongodb的windows服务的安装
date: 2018-1-25
tags: [数据库, mongodb]
---

> 这篇文章是写给媳妇看的。


## 1. 安装文件

官方网站 [http://www.mongodb.org/downloads](http://www.mongodb.org/downloads), 选择对应系统的文件下载。

## 2.配置环境变量

如果没有更改安装目录，mongodb可能安装在 `C:\Program Files\MongoDB`。

这样的话，就将 `C:\Program Files\MongoDB\Server\3.6\bin`添加到环境变量中。

如果不是的话，只要知道`mongo.exe` 和 `mongod.exe`所在路径，并添加到环境变量中即可。重新打开`cmd`，输入命令`mongod`，提示不是“'mongod' 不是内部或外部命令，也不是可运行的程序或批处理文件。”，就可以认为安装成功。

<!--more-->

## 3.启动服务

在C盘根目录创建文件夹`data`,创建子文件夹`db`和`log`。

- 命令行启动服务：

    mongod --dbpath=C:\data

就可以启动服务，但是这命令行不能关闭，否则服务就会关闭。

- windows服务启动。

以 **管理员** 权限打开命令行（windows10可以按win+x和a启动）执行以下命令：

```
mongod --dbpath=C:\data\db --logpath=C:\data\log\mongodb.log --install --serviceName "mongo" --logappend --directoryperdb
```

然后启动服务：

```
net start mongodb
```

会出现 “Mongo DB 服务已经启动成功”的提示。

## 4.连接

输入命令`mongo` 即可实现连接。


## 5.图形化管理工具

mongodb3.6版本(以前的版本我就不知道了)安装完成之后，会自动安装一个图形化管理工具：`MongoDB Compass Community`

![TIM图片20180126094759.png](https://i.loli.net/2018/01/26/5a6a8914df1f0.png)
![TIM图片20180126094827.png](https://i.loli.net/2018/01/26/5a6a89155eeaf.png)
