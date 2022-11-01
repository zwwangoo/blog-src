---
title: 使用Github的Actions自动构建个人博客
date: 2022-11-02
tags: [github,blog]
---

个人博客之前使用的是 hexo+travis自动构建github page，但是由于长时间未更新内容，travis也进行了迁移和收费等变更，导致更新blog-src的博客文章之后，不能自动生成静态网页并推送到zwwangoo.github.io这个仓库，正巧发现github自带的Actions功能已经能满足我的需求，那么简单折腾了一下，现在就不需要借用travis啦。
之前的部署方式：[hexo+travis自动构建github page](https://zwwangoo.github.io/2018/04/21/hexo+travis自动构建github%20page/)

本文参考: [Github Actions 通过 SSH 自动备份到代码托管网站](https://blog.csdn.net/z_johnny/article/details/104061608)

## 前提条件：

拥有github帐号，并创建了相关仓库，这里以我个人的为例：
- `zwwangoo.github.io` github page仓库，这里存放编译生成的静态网站相关信息，访问https://zwwangoo.github.io看到的博客就是由此提供。
- `blog-src` 博客源码存放的地方，也就是hexo相关的模板。

<!--more-->

## 秘钥准备

为了方便运行GitHub Actions时登录GitHub账号，我们使用SSH方式登录。就是要把设备的私钥交给GitHub Actions，公钥交给GitHub，需要去Settings里去配置。

使用ssh-keygen生成一组公私秘钥对：

```
ssh-keygen -t rsa -C "zwwangoo@gmail.com"
```

- 生成的密钥一定要保存好了，最重要的是私钥，配置公钥，应该已经配好，不然如何上到的项目资源，配置路径：github网站–>Settings–>SSH and GPG keys
- 配置仓库私钥，blog私有仓库的 Settings --> Secrets --> Actions 里添加私钥Secrets，以名称`token_Private_Keys`命名，输入对应的值，这个值就是私钥！

## 配置GitHub Actions

GitHub Actions 有一些自己的术语。

- workflow （工作流程）：持续集成一次运行的过程，就是一个 workflow。
- job （任务）：一个 workflow 由一个或多个 jobs 构成，含义是一次持续集成的运行，可以完成多个任务。
- step（步骤）：每个 job 由多个 step 构成，一步步完成。
- action （动作）：每个 step 可以依次执行一个或多个命令（action）

在blog仓库选择Actions --> "Publish Node.js Package" 点击Configure之后按照以下内容填写：

```
name: Node.js Package

on:
  push:
    branches:
    - main
    - master

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 16
      - name: Setup Git Infomation
        run: | 
          git config --global user.name 'zwwangoo'
          git config --global user.email 'w_angzhiwen@163.com' 
        
      - name: Setup SSH Private Key
        env:
          token_Private_Keys: ${{ secrets.token_Private_Keys }}
        run: |
          mkdir -p ~/.ssh/
          echo "$token_Private_Keys" > ~/.ssh/id_rsa 
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
      - name: npm install, and push
        run: |
          npm install hexo hexo-cli
          node_modules/hexo/bin/hexo d -g
```

注意修改你的git config 的 user.name 和 user.email
提交文件