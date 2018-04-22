---
title: hexo+travis自动构建github page
date: 2018-4-22
tags: [hexo, travis]
---

将近一天的折腾，终于ok了，实现了如题的新玩法。这里做一个简单的记录，记录自己踏过的坑，其实还主要是ssh-key的问题。主要参考[这里](https://gythialy.github.io/travis-add-ssh-key/)

## 前提条件：

- 拥有github帐号，并创建了相关仓库，这里以我个人的为例：
  + [suadminwen.github.io](https://github.com/suadminwen/suadminwen.github.io)  github page仓库，这里存放编译生成的静态网站相关信息，访问[https://suadminwen.github.io](https://suadminwen.github.io)看到的博客就是由此提供。
  + [blog-src](https://github.com/suadminwen/blog-src) 博客源码存放的地方，也就是hexo相关的模板。
- 拥有与github关联的travis帐号。

<!--more-->

## 开启对 项目的集成并进入设置页面

![例图](/blog-img/2018042201.png)

## 生成ssh-key

- ssh-keygen -t rsa -b 4096 -C "<your_email>" -f github_deploy_key -N ''

> 注： 这里使用 github_deploy_key 作为存储的名字

**这里创建的ssh-key是给travis提交代码的时候用的，换句话说也就是github page 那个仓库用的key！这里被坑了很久，一直把key添到博客源码的地方。**

这会生成两个文件

- 公钥 github_deploy_key.pub
- 私钥 github_deploy_key

> 如果是做为项目的部署公钥，需要添加到项目中，以 GitHub 为例:
> 用文本编辑器打开“github_deploy_key.pub”文件，复制内容到剪贴板。
> 打开 [https://github.com/settings/ssh](https://github.com/settings/ssh) ，点击 Add SSH Key 按钮，粘贴进去保存即可。

## 安装 The Travis Client

- 首先确保已经安装好 Ruby(1.9.3+)，官方推荐 2.0.0

- 执行 `gem install travis -v 1.8.8 --no-rdoc --no-ri` 安装 `travis client`

- 检查安装是否正确 `ravis version`

## 加密 SSH key

GitHub 项目，建议先通过 `travis login` 登录，然后再通过 `travis encrypt-file github_deploy_key -add` 加密，
travis Client 会自动更新 `.traivs.yaml`并且在 travis 中自动添加变量 `encrypted_XXXXXXXXXXXX_key` 和 `encrypted_XXXXXXXXXXXX_iv`

> 注： 这里的`github_deploy_key`是上面的私钥！

## 修改 .travis.yaml

在源码根目录下创建,travis目录，将创建的`github_deploy_key.enc`文件移至该目录下，并在该目录下创建`ssh_config`文件。

- 修改.travis.yaml:

在 `before_install` 添加如下内容：
```
before_install:
- openssl aes-256-cbc -K $encrypted_a7d17a00ff1b_key -iv $encrypted_a7d17a00ff1b_iv
  -in .travis/github_deploy_key.enc -out ~/.ssh/github_deploy_key -d
- chmod 600 ~/.ssh/github_deploy_key
- eval $(ssh-agent)
- ssh-add ~/.ssh/github_deploy_key
- cp .travis/ssh_config ~/.ssh/config
- git config --global user.name 'gythialy'
- git config --global user.email 'gythialy@users.noreply.github.com'
```

> 注： 步骤如下：通过openssl 解密文件并输出到 ~/.ssh/github_deploy_key 中；设定 ~/.ssh/github_deploy_key 文件权限并添加到 ssh-agent 中

- 添加`ssh_config`内容，主要是防止首次连接的时候，会弹出提示。如果有其他的地址，参考此设置即可。

```
Host github.com
    User git
    StrictHostKeyChecking no
    IdentityFile ~/.ssh/github_deploy_key
    IdentitiesOnly yes
Host gitcafe.com
    User git
    StrictHostKeyChecking no
    IdentityFile ~/.ssh/github_deploy_key
    IdentitiesOnly yes
Host git.coding.net
    User git
    StrictHostKeyChecking no
    IdentityFile ~/.ssh/github_deploy_key
    IdentitiesOnly yes
```

至此，就成功在 travis 中添加了SSH密钥且能建立链接。