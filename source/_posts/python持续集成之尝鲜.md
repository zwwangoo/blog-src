---
title: python持续集成之尝鲜
date: 2018-2-10
tags: [python, TDD]
---

**驱动测试** 和 **持续集成** 的概念正在一点一点的“腐蚀”我的思想，不去尝尝鲜便发现坐立不安。这不，这日，来了。环境是用两台虚拟机，一台用作gitlab的服务器，另外一台为jenkins的服务器，这就happy起来啦！

以下是正文，学习参考自：[这里](http://www.cnblogs.com/beer/p/5083653.html)
---

**持续集成（Continuous integration）** 是一种软件开发实践，即团队开发成员经常集成它们的工作，通过每个成员每天至少集成一次，也就意味着每天可能会发生多次集成。每次集成都通过自动化的构建（包括编译，发布，自动化测试）来验证，从而尽早地发现集成错误.

<!--more-->

## 持续集成的角色划分：

- 开发人员
  + 编写开发相关代码
- 测试人员
  + 设计自动化测试用例
  + 编写自动化测试相关代码
- 发布人员
  + 设置自动化发布方案
  + 编写自动化发布相关代码
- 运维人员
  + 设置自动化运维方案
  + 编写自动化运维相关代码

持续集成的典型的工具就是开源系统 Jenkins。

## 后期迭代工作流：

- 开发人员向git服务器指定分支提交了新的代码
- git服务器的 webhook 接收到提交事件后向 Jenkins 服务器指定接口发起请求，执行构建脚本
- Jenkins 服务器顺序执行构建脚本
  + 从git服务器上面同步代码
  + 执行自动构建脚本，生成交付物
  + 自动搭建测试环境
- Jenkins 执行自动化测试脚本
- Jenkins 向 自动化发布 系统发起请求
- 自动化发布系统 进行自动灰度发布
- 触发 自动化测试系统
- 逐步全网发布

## 实际操作：
![这是本地图片测试，图床一段时间之后就失效了](/blog-img/2018021001.png)
