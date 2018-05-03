---
title: 使用git在服务器上部署git仓库并实现提交代码时同步代码到生产环境
date: 2017-10-27
tags: [git, 服务器]
---

最近由于需要对正在运行的系统进行新功能添加，本来是可以通过github进行代码维护，但是由于这个项目涉及一些问题，目前还不能开源，所以只能是手动覆盖bug文件，生产环境上的代码反而是最新的了。

之前有个思路，就是将git仓库部署到生产环境所在的服务器中，今天做了一下尝试，效果还不错，特意再次做出总结。具体实现的目标就是：

<!-- more -->

使用git在服务器上部署git仓库并实现提交代码时同步代码到生产环境，参考[这里棒棒哒](https://my.oschina.net/u/190049/blog/488588).废话少说，进入正题。

首先，需要在服务器上安装git，这里就不再罗嗦，不会装的，出门左转[点击这里](https://git-scm.com/)。

以下代码命令和代码实例中说明一下几点：

- 所有服务器上的操作，为了避免遇到权限的问题，是直接进入`su`超级用户下的操作。
- 创建的项目名为`my_site`。
- 在服务器上，我选择将仓库创建在`/var/`下，生产环境项目代码放在`/projects/my_site/`。阅读者可以自行更改，但是需要记住更改后的路径。
- 为了隐私，实例的服务器ip用`serverhost`代替，用户用user代替。

## 在服务器上创建git仓库

我选择在`/var/`下创建：

    mkdir git  && cd git
    mkdir my_site.git && cd my_site.git
    git init --bare

`--bare`的意思是，该文件夹是我们的代码仓库，它将不会放源代码而只是做版本控制。

## 创建钩子

将会使用post-receive钩子，更多相关的信息可以参考官方文档。

    ls

可以看到hooks已创建，而且里面也有各种钩子的样例。

![11.jpg](http://upload-images.jianshu.io/upload_images/3248493-23f7a5b78a55b7f6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

创建我们自己的post-receive：

```shell
cd hooks
vim post-receive
```
在`post-receive`中需要同步的语句

```
#!/bin/sh
git --work-tree=/projects/my_site/ --git-dir=/var/git/your_site.git checkout -f
```

编辑完成之后保存。

将git仓库设成可读写的：

    chmod 777 -R *

如果生产环境项目所在目录`/projects/my_site/`不存在，要记得创建，同时让其的权限成为任何人都可读写！

```
mkdir /projects/my_site/
cd /projects/ && chmod 777 -R *
```

服务器上的仓库和同步钩子配置到此完毕，下面需要我们在本地编写项目并上传至服务器仓库。

## 本地项目推送

一般情况是你**已经有**了自己的git项目了，那么只需要添加仓库地址就行了。

    git remote add server ssh://user@serverhost/var/git/my_site.git

'server'只是这个远程连接的名称，你可以同时有多个远程连接，每次push的时候指定名称即可将代码上传到不同的仓库。

如果你本地还没有项目代码：

```
cd my_site
git init
git add *
git commit -m "commit"
git remote add server ssh://user@serverhost/var/git/my_site.git
git push server master
```
master指定的是master分支，如果你有其他分支也可以push其他分支。

补充：

我们也可以从git仓库中clone代码到本地：

    git clone ssh://user@serverhost/var/git/my_site.git

--