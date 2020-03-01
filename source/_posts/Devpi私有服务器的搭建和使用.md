---
title: Devpi私有服务器的搭建和使用
date: 2020-02-24
tags: [Python, devpi,pip] 
---

## 介绍
Python私有包的发布，可以使用Pypi、Devpi等可以自己搭建的服务器，相对与Pypi Server，Devpi不仅提供私有包的发布，还有以下特色：

- 支持本地缓存，可以做到公司内网加速的效果。
- 支持[Sphinx](http://sphinx-doc.org/)文档。

- 提供多索引，多索引之间还可以继承，这在维护多版本系统上十分有用。
- 支持集群部署，支持一台或多台服务器部署，来加速访问。还支持通过 json 接口，实时监控集群的状态。
- 支持导入导出功能。
- 支持给索引设置 Jenkins 触发器，可以使用 tox 自动测试上传的包。
- 使用插件可以完成Web界面的访问控制，增加私有包的安全。
## 自建devpi服务
devpi服务包含三个组件：

- devpi-server 是 devpi server 核心组件，提供镜像与缓存功能
- devpi-web 提供Web界面和查询功能
- devpi-client 命令行工具, 提供包上传等与服务器交互的功能

额外安装的组件：

- devpi-lockdown在nginx的帮助下实现对Web界面添加访问控制的功能

<!--more-->

已经编写好的dockerfile，可以直接使用([https://github.com/suAdminWen/mydocker/tree/master/devpi-server](https://github.com/suAdminWen/mydocker/tree/master/devpi-server))，包含以上四个组件。

- 访问控制


![图片.png](/blog-img/2020022401.png)

- 输入帐号密码登录之后


![图片.png](/blog-img/2020022402.png)
### 索引规划
![图片.png](/blog-img/2020022403.png)
root用户是初始化用户，不可删除和修改，root/pypi直接从公有镜像服务查询。
devuser是自己创建的用户，该用户下有两个索引：

- staging 对应的是生产环境发布的包，不可删除和修改，基础是root/pypi
- dev 对应的是开发环境发布的包，可以进行维护。基础是devuser/staging

从devuser/dev索引上下载包时，当包不存在，首先会从devuser/staging索引查询，查询不到时，会在root/pypi上查询，再查询不到时会转向root/pypi对应的公有服务查询。


## 初始化设置
本地的操作是使用devpi-client工具。
安装devpi-client：`pip install -U devpi-client`
首先是创建连接 `devpi use http://devpi.xxxxx.com/` 
### 创建用户
使用root用户登录，初始为设置密码：

```bash
devpi login root --password=
# 修改root用户密码为123
devpi user -m root password=123 
# 创建新用户devuser并设定密码为123
devpi user -c devuser password=123
# 新用户登录
devpi login devuser --password=123
# 退出登录
devpi logoff
```

### 创建索引
登录之后才可以进行操作

```bash
# 创建staging索引
devpi index -c staging bases=root/pypi
# 使用staging索引
devpi use devuser/staging

# devpi index dev mirror_whitelist="*"
```

## 上传私有包
上传包使用的是 `devpi upload` 命令，需要在 setup.py 所在目录下执行，这个命令有两个常用的参数：

- `--with-docs` 参数，连带docs文件一块上传，支持 sphinx 创建的文档，需要 docs 目录和 setup.py 在同个目录下， `pip install sphinx`
- `--formats bdist_wheel` 参数，上传wheel格式的包，需要安装wheel库， `pip install wheel`

示例：

```bash
devpi use http://devpi.xxxxxx.com/
devpi login devuser --password=123
devpi use devuser/dev
devpi upload --formats bdist_wheel
```
`devpi push` 命令是将包从一个索引推送到另外一个索引，例如将包example推送到devuser/staging
```bash
devpi push example==1.0 devuser/staging
```

## pip下载包
### 直接在终端使用
无访问限制：

```bash
pip install -i "http://[host]/devuser/dev/+simple/" [package] --trusted-host [host]
```
有访问限制：
```bash
pip install -i "http://[user]:[password]@[host]/devuser/dev/+simple/" [package] --trusted-host [host]
```
### 配置~/.pip/pip.conf
如果有访问限制，http://后要使用 `http://[user]:[password]@[host]`
```bash
[global]
timeout = 60
index-url = http://devpi.example.com/devuser/dev/+simple/
[install]
trusted-host = devpi.example.com
```

