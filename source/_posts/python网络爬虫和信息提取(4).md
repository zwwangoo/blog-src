---
title: python网络爬虫和信息提取(4)
date: 2017-12-19
tags: [python, 网络爬虫]
---

## Scrapy框架的使用和实战

Scrapy是一个功能强大的python爬虫框架！

爬虫框架：
- 是实现爬虫功能的一个软件结构和功能组件集合。
- 是一个半成品，能够帮助用户实现专业网络爬虫。

### 安装

    pip install scrapy

### Scrapy常用命令

| 命令 | 说明 | 格式 |
| :------------- | :------------- |
| startproject | 创建一个新工程 | `scrapy startproject <name> [dir]` |
| genspider | 创建一个爬虫 | `scrapy genspider [options] <name> <domain>` |
| settings | 获取爬虫配置信息 | scrapy settings [options] |
| crawl | 运行一个爬虫 | scrapy crawl <spider> |
| list | 列出工程中所有爬虫 | scrapy list |
| shell | 启动URL调试命令行 | scrapy shell [url] |


## requests vs Scrapy

相同点：

- 两者都可以进行页面请求和爬取，python爬虫的两个重要技术路线。
- 两者可用性都好，文档丰富，入门简单。
- 两者都没有处理js、提交表单、对应验证码等功能（可扩展）。

不同点:

| requests | scrapy |
| :------------- | :------------- |
| 页面级爬虫 | 网站级爬虫 |
| 功能库 | 框架 |
| 并发性考虑不足，性能较差 | 并发性好，性能较高 |
| 重点在于页面下载 | 重点在于爬虫结构 |
| 定制灵活 | 一般定制灵活，深度定制困难 |
| 上手十分简单 | 入门稍难 |


选用哪个技术路线开发爬虫

- 非常小的需求，requests库
- 不太小的需求，scrapy框架
- 定制程度很好的需求(不考虑规模),自搭框架。requests > Scrapy。
