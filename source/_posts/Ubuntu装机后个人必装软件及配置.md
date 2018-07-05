---
title: Ubuntu装机后个人必装软件及配置
date: 2018-06-19
tags: [Ubuntu]
---

# Ubuntu 16 版本  

## shadowsocks-qt5

```bash
sudo add-apt-repository ppa:hzwhuang/ss-qt5

sudo apt update

sudo apt install shadowsocks-qt5 -y
```

<!--more-->

## ssr-qt5与全局代理

[参考这里](https://www.litcc.com/2016/12/29/Ubuntu16-shadowsocks-pac/index.html)

```bash
sudo pip install genpac
pip install --upgrade genpac
```

### 下载规则

https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt 保存到本地gfwlist.txt

在fwlist.txt 目录下，执行以下命令：

```bash
genpac --pac-proxy "SOCKS5 127.0.0.1:1080" --gfwlist-proxy="SOCKS5 127.0.0.1:1080" --gfwlist-local=gfwlist.txt --output="autoproxy.pac"
```

### 设置全局代理

点击：System settings > Network > Network Proxy，选择 Method 为 Automatic，设置 Configuration URL 为 autoproxy.pac 文件的路径，点击 Apply System Wide。
格式如：file:///home/{user}/autoproxy.pac

file:///home/ouou/Downloads/autoproxy.pac

## unity-tweak-tool

```bash
sudo apt-get install unity-tweak-tool
```

## google-chrome

```bash

sudo wget https://repo.fdzh.org/chrome/google-chrome.list -P /etc/apt/sources.list.d/

wget -q -O - https://dl.google.com/linux/linux_signing_key.pub  | sudo apt-key add -

sudo apt-get update

sudo apt-get install google-chrome-stable

#  /usr/bin/google-chrome-stable
```

## 网易云音乐

下载地址：[http://d1.music.126.net/dmusic/netease-cloud-music_1.1.0_amd64_ubuntu.deb](http://d1.music.126.net/dmusic/netease-cloud-music_1.1.0_amd64_ubuntu.deb)

## WPS for Linux

官网下载安装包[http://kdl.cc.ksosoft.com/wps-community/download/6634/wps-office_10.1.0.6634_amd64.deb](http://kdl.cc.ksosoft.com/wps-community/download/6634/wps-office_10.1.0.6634_amd64.de)

下载完成之后，进行安装

```bash
sudo dpkg -i wps-office_10.1.0.6634_amd64.deb

# 以下是卸载libreOffice
sudo apt-get remove libreoffice-common

sudo apt-get remove unity-webapps-common

sudo apt autoremove
```

### 字体缺失问题

下载字体包：

[https://pan.baidu.com/s/1eS6xIzo](https://pan.baidu.com/s/1eS6xIzo)

下载之后解包，然后

```bash
sudo cp mtextra.ttf  symbol.ttf  WEBDINGS.TTF  wingding.ttf  WINGDNG2.ttf  WINGDNG3.ttf  /usr/share/fonts
```

重启wps 即可。

## 安装MAC字体

```bash
wget -O mac-fonts.zip http://drive.noobslab.com/data/Mac-14.04/macfonts.zip

sudo unzip mac-fonts.zip -d /usr/share/fonts; rm mac-fonts.zip

sudo fc-cache -f -v
```

## 状态栏显示网速

```bash
sudo add-apt-repository ppa:nilarimogard/webupd8
sudo apt-get update
sudo apt-get install indicator-netspeed
```

之后把 `indicator-netspend` 加入到自动的脚本中去。[参考](https://blog.csdn.net/sinat_36219858/article/details/61195905)


