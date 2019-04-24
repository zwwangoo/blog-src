---
title: Confluence安装在Linux上
date: 2019-04-24
tags: [Ubuntu, 敏捷开发]
---

Confluence是一个适用于团队协作的文档管理软件，使用java开发的，功能方面类似Wiki，但是功能比Wiki强大。在安全性方面，Confluence 具有完善和精细的权限控制，可以很好地控制用户在 Wiki 中创建、编辑内容和添加注释。Confluence 直观易于使用，您不需要学习任何新的语言就可以使用它，用所见即所得的方式在 Confluence 中添加内容。

虽然Confluence是收费的，但是Atlassian公司将其使用门槛降低了，最低设置了10人版每年10$的授权费，企业可以根据具体的使用人数进行[购买授权](https://cn.atlassian.com/software/confluence/pricing?tab=self-managed)。

个人学习使用，这里根据网上的资源提供了一个破解的方法，仅供学习参考。

## 下载安装

去官网下载最新的安装包（[传送门](https://www.atlassian.com/software/confluence/download)）。官网下载可能比较慢，可以通过下面的链接下载(6.15.2 Linux版本)，顺带下载破解的工具包。

破解包: https://pan.baidu.com/s/1SYJ_nWfNkH0AlgBjqcVBsA (提取码: pava)

<!--more-->

### JDK环境安装

### 安装mysql

这里使用了外部的mysql数据库：

```
sudo apt update
sudo apt install mysql-server mysql-client
```

创建对应的数据库并授权指定用户

```
mysql -u root -p
mysql> create database confluence character SET utf8 COLLATE utf8_bin;
mysql> SET GLOBAL tx_isolation='READ-COMMITTED';
mysql> grant all on confluence.* to confluence@"%" identified by "123456";
mysql> FLUSH PRIVILEGES;
```

mysql的其他安装配置可以参考[Mysql数据库配置]

### 安装Confluence

给atlassian-confluence-6.15.2-x64.bin添加运行的权限：

```
sudo chmod +x atlassian-confluence-6.15.2-x64.bin
```

运行上述文件：

```
sudo ./atlassian-confluence-6.15.2-x64.bin
```

![1555988876289](https://i.loli.net/2019/04/23/5cbea0d5b9574.png)

先后输入o，1，i三个指令，需要注意的是Confluence的安装路径。安装过程中，没有选择启动，这里补充一些Confluence的启动服务和关闭服务：

```
# 关闭服务
sudo service confluence stop
# 启动服务
sudo service confluence start
```

Confluence的卸载：

```
sudo /opt/atlassian/confluence/uninstall
# 手动删除两个路径
sudo rm -rf /opt/atlassian /var/atlassian
```

## 破解Confluence

安装完成之后，启动Confluence：

```
sudo service confluence start
```

启动完成之后，可以在浏览器http://127.0.0.1:8090访问到服务，记下Server ID。

![1555989749457.png](https://i.loli.net/2019/04/23/5cbe9e94c34b9.png)

![1555989764220.png](https://i.loli.net/2019/04/23/5cbe9e94bf903.png)

![1555989782771](https://i.loli.net/2019/04/23/5cbe9e94a9e53.png)

然后可以停止服务：

```
sudo service confluence stop
```

下载工具包，解压。将`/opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-decoder-v2-3.4.1.jar` 复制到方便的位置，我这里放到了home下。

```
sudo cp /opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-decoder-v2-3.4.1.jar /home/
```

然后进入工具包：

```
# 将工具包中mysql-connector-java-5.1.32-bin.jar复制到Confluence的依赖文件夹下
sudo cp mysql-connector-java-5.1.32-bin.jar /opt/atlassian/confluence/confluence/WEB-INF/lib

# 用原版复制出来的atlassian-extras-decoder-v2-3.4.1.jar 包替换该目录下 atlassian-extras-2.4.jar
cd confluence5.1-crack
sudo cp /home/atlassian-extras-decoder-v2-3.4.1.jar atlassian-extras-2.4.jar
cd iNViSiBLE
sudo bash ./keygen.sh
```

**注：必须是在图形界面下，因为这个运行需要图形。如果没有图形，那么就会报错。**

首先填入Server ID，在图形工具点击`.patch`选择`atlassian-extras-2.4.jar`文件。

![1555989833935](https://i.loli.net/2019/04/23/5cbe9e94a2bdf.png)

![1555989867280](https://i.loli.net/2019/04/23/5cbe9e94921fc.png)

然后点击`.gen`生成key，复制所得的key。

![1555989885366](https://i.loli.net/2019/04/23/5cbea0a94bece.png)

在原本的atlassian-extras-2.4.jar所在的目录，多了一个atlassian-extras-2.4.bak为文件，不用管该文件。需要将atlassian-extras-2.4.jar文件替换成Confluence中的atlassian-extras-decoder-v2-3.4.1.jar，也就是复制出来的那个文件。

```
cd ..
sudo cp atlassian-extras-2.4.jar /opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-decoder-v2-3.4.1.jar
```

![1555989913161](https://i.loli.net/2019/04/23/5cbe9e94be989.png)

点击下一步，就可以快乐的玩耍啦：

![1555991064801](https://i.loli.net/2019/04/23/5cbea05bf418c.png)

设置数据库的时间较长，请耐心等待。

![1555993565787](https://i.loli.net/2019/04/23/5cbea05c02448.png)



参考

- [confluence安装破解](<https://www.jianshu.com/p/d621c5eec1c8>)
- [linux 破解版confluence安装](https://www.cnblogs.com/wspblog/p/4750128.html)
