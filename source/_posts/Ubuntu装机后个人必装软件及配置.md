---
title: Ubuntu装机后个人比装软件及配置
date: 2018-06-19
tags: [Ubuntu]
---

# Ubuntu 16 

## shadowsocks-qt5

```
sudo add-apt-repository ppa:hzwhuang/ss-qt5

sudo apt update

sudo apt install shadowsocks-qt5 -y
```

## ssr-qt5与全局代理

[参考这里](https://www.litcc.com/2016/12/29/Ubuntu16-shadowsocks-pac/index.html)

```
sudo pip install genpac
pip install --upgrade genpac
```

<!--more-->

### 下载规则

https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt 保存到本地gfwlist.txt

在fwlist.txt 目录下，执行以下命令：

```
genpac --pac-proxy "SOCKS5 127.0.0.1:1080" --gfwlist-proxy="SOCKS5 127.0.0.1:1080" --gfwlist-local=gfwlist.txt --output="autoproxy.pac"
```

### 设置全局代理

点击：System settings > Network > Network Proxy，选择 Method 为 Automatic，设置 Configuration URL 为 autoproxy.pac 文件的路径，点击 Apply System Wide。
格式如：file:///home/{user}/autoproxy.pac

file:///home/ouou/Downloads/autoproxy.pac

## unity-tweak-tool

```
sudo apt-get install unity-tweak-tool
```

## google-chrome

```

sudo wget https://repo.fdzh.org/chrome/google-chrome.list -P /etc/apt/sources.list.d/

wget -q -O - https://dl.google.com/linux/linux_signing_key.pub  | sudo apt-key add -

sudo apt-get update

sudo apt-get install google-chrome-stable

#  /usr/bin/google-chrome-stable
```
