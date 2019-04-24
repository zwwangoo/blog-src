---
title: 敏捷开发之Jenkins的部署和基本使用
date: 2019-04-24
tags: [敏捷开发, Ubuntu]
---

## java环境安装

Jenkins新版本已经不支持jdk7及以下版本，这里安装jdk8。

更新软件包列表

```
sudo apt update
```

### Ubuntu16 安装 jdk8

```bash
sudo apt install openjdk-8-jdk
```

查看Java版本，看看是否安装成功

```
java -version
```

<!--more-->


![1555927916172](https://i.loli.net/2019/04/23/5cbea14a55e6f.png)

多版本jdk时，版本之间的切换

```
sudo update-alternatives --config java
```

前面带星号的是当前正在使用的Java版本，键入编号选择使用哪个版本。

![1555927981317](https://i.loli.net/2019/04/23/5cbea14a71aa0.png)

### Ubuntu16 安装jdk7

这里补充一点Ubuntu系统中jdk7的安装（为什么使用jdk7，历史遗留问题。）

jdk7已经不支持从源中下载，所以需要手动下载安装包进行安装。

- 首先需要依次下载并安装 jdk7 以及 jdk7 依赖的类库：

[fontconfig-config](https://packages.debian.org/sid/fontconfig-config)

[libfontconfig1](https://packages.debian.org/sid/libfontconfig1)

[libjpeg62-turbo](https://packages.debian.org/sid/libjpeg62-turbo)

[openjdk-7-jre-headless](https://packages.debian.org/experimental/openjdk-7-jre-headless)

[openjdk-7-jre](https://packages.debian.org/experimental/openjdk-7-jre)

[openjdk-7-jdk](https://packages.debian.org/experimental/openjdk-7-jdk)

- 下载的时候要选择和自己平台匹配的 deb 包，我安装的是 64 位的 Ubuntu 系统，所以我下载的是 amd64 的 deb 包。然后把下载的这六个 deb 文件放在一个空文件夹里面，执行：

```
sudo dpkg -i *.deb
```

如果出现其他依赖的问题，执行：

```
sudo apt install -f
```

- 执行完之后 jdk 7 就安装成功了。

- 执行 java -version 看一下当前版本号，如果是 jdk-1.7 那么你就成功了。

## Jenkins安装

我们从Jenkins官方网站<https://jenkins.io/>下载最新的war包。虽然Jenkins提供了Windows、Linux、OS X等各种安装程序，但是，这些安装程序都没有war包好使。只需要运行命令：

```
java -jar jenkins.war
```

Jenkins就启动成功了！它的war包自带Jetty服务器，剩下的工作我们全部在浏览器中进行。默认使用8080端口，如果想使用其他端口命令改成：

```
java -jar jenkins.war --httpPort=9001
```

第一次启动Jenkins时，出于安全考虑，Jenkins会自动生成一个随机的按照口令。

![1555998670165](/home/wen/Pictures/typora/1555998670165.png)

注意控制台输出的口令，复制下来，然后在浏览器输入：[http://127.0.0.1:8080](http://127.0.0.1:8080)

粘贴口令，进入安装界面，如果执行默认的安装，Jenkins就自动配置好了Maven、git等常用插件。默认安装插件的耗时较长，要耐心等待。最后，创建一个admin用户，完成安装。

## 创建Linux服务

因为我们不想每次登录到Linux去启动Jenkins，也不想写脚本来启动服务。推荐安装JDK后，配合supervisor，把Jenkins直接变成一个服务。

```
#/etc/supervisor/conf.d/ci.conf
[program:ci]                                                                                   command=java -jar /usr/lib/jenkins.war --httpPort=9001
user=ubuntu
autostart=true
autorestart=true
startsecs=30
startretries=5
```

## Jenkins的使用

在Jenkins首页选择 “新建任务”，输入名字，选择“构建一个自由风格的软件项目”

![1555984239991](https://i.loli.net/2019/04/23/5cbea14aa3ef5.png)

在配置页中，源码管理选择Git，填入地址：

![1555984798267](https://i.loli.net/2019/04/23/5cbea14a7e0e0.png)

默认使用master分支。如果需要口令，在Credentials中添加用户名/口令，或者使用SSH Key。

构建触发器指定了触发一次构建的条件。推荐使用最简单的配置“Poll SCM”，它的意思是，定时检查版本库，发现有新的提交就触发构建。这种方式对git、SVN等所有版本管理系统都是通用的。

我们在日程表中填入：

```
* * * * *
```

![1555984850539](https://i.loli.net/2019/04/23/5cbea14a87491.png)

表示每分钟检查一次。如果你觉得太频繁，可以改成“每3分钟检查一次”：

```
*/3 * * * *
```

在演示项目中，使用了执行Shell构建的方法，所以在“构建”中选择“执行shell”。

这里的shell是构建项目的过程。可以简单的写一个测试一下：

```
echo "Hello ------------------"
```

点击保存，就可以执行自动化构建了。

![1555985221427](https://i.loli.net/2019/04/23/5cbea14a971ce.png)可以在Console Output中看到控制台详细输出，便于出错排查：

![1555985266485](https://i.loli.net/2019/04/23/5cbea14ab26a5.png)

这里为正确的输出。构建完成。

## 关于Docker

这里建议的是使用Docker来构建集成环境，Docker和jenkins都在主机的环境中，gitlab可以使用Docker部署。Nginx代理各个服务。



参考

- [在Ubuntu 18.04.1系统中安装Jdk 7（openjdk-7-jdk](<https://ywnz.com/linuxjc/2734.html>)
- [使用Jenkins进行持续集成](<https://www.liaoxuefeng.com/article/001463233913442cdb2d1bd1b1b42e3b0b29eb1ba736c5e000>)
