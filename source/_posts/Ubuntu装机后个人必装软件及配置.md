---
title: Ubuntu装机后个人必装软件及配置
date: 2018-06-19
tags: [Ubuntu]
---

# Ubuntu 18 版本  

## google-chrome

```bash

sudo wget https://repo.fdzh.org/chrome/google-chrome.list -P /etc/apt/sources.list.d/

wget -q -O - https://dl.google.com/linux/linux_signing_key.pub  | sudo apt-key add -

sudo apt-get update

sudo apt-get install google-chrome-stable
```

安装完成之后，命令行启动

`/usr/bin/google-chrome-stable`

启动完成之后锁定到启动器即可。

## shadowsocks-qt5

```bash
sudo add-apt-repository ppa:hzwhuang/ss-qt5

sudo apt update

sudo apt install shadowsocks-qt5
```

Ubuntu18及以上版本会在`add-apt-repository`之后会报错，需要手动去改源中的版本信息：

打开文件：

```
sudo vi /etc/apt/sources.list.d/hzwhuang-ubuntu-ss-qt5-bionic.list
```

将下面`bionic`改成 `xenial`:

```
deb-src http://ppa.launchpad.net/hzwhuang/ss-qt5/ubuntu xenial main
```

## ssr-qt5与全局代理

**注意，Ubuntu 18及以上版本，以下方式添加全局代理已经无效。目前的做法就是，在设置网络中，“网络代理”选择手动，填入相应的代理IP和端口。然后登录谷歌，同步插件SwitchyOmega，进行代理。成功之后再禁用设置网络中的网络代理。**

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

file:///home/wen/Downloads/autoproxy.pac

## gnome-tweak-tool

界面优化工具，可以自定义系统主题、图标、锁屏界面内容、字体，管理插件等内容

```bash
sudo apt-get install gnome-tweak-tool
```

<!--more-->

## 网易云音乐

[官网搬运V1.1.0](http://d1.music.126.net/dmusic/netease-cloud-music_1.1.0_amd64_ubuntu.deb)

[官网搬运V1.2.1](http://d1.music.126.net/dmusic/netease-cloud-music_1.2.1_amd64_ubuntu_20190428.deb)

下载完成之后，在文件所在目录：

```
sudo dpkg -i netease-cloud-music_1.1.0_amd64_ubuntu16.04.deb
```

## WPS for Linux

[官网搬运](https://wdl1.cache.wps.cn/wps/download/ep/Linux2019/8722/wps-office_11.1.0.8722_amd64.deb)

下载完成之后，进行安装

```bash
sudo dpkg -i wps-office_11.1.0.8722_amd64.deb
```

### 以下是卸载libreOffice

```
sudo apt remove libreoffice-common

sudo apt remove unity-webapps-common

sudo apt autoremove
```

### 字体缺失问题

下载字体包：

[百度网盘](https://pan.baidu.com/s/1eS6xIzo)
[谷歌云端硬盘共享](https://drive.google.com/file/d/1K2G1cVWYEUeV4IuciRzKAqlHxlzFr9HI/view?usp=sharing)

下载之后解包，然后

```bash
sudo mv mtextra.ttf  symbol.ttf  WEBDINGS.TTF  wingding.ttf  WINGDNG2.ttf  WINGDNG3.ttf  /usr/share/fonts
sudo fc-cache -f -v
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

## 搜狗输入法

一顿操作猛如虎：

```
wget http://cdn2.ime.sogou.com/dl/index/1524572264/sogoupinyin_2.2.0.0108_amd64.deb

sudo dpkg -i sogoupinyin_2.2.0.0108_amd64.deb
sudo apt install -f 
sudo dpkg -i sogoupinyin_2.2.0.0108_amd64.deb
sudo rm sogoupinyin_2.2.0.0108_amd64.deb

```
安装完成之后，好像需要重启电脑一下，然后就可以通过 `CTR+空格键` 切换输入法了。

## 美化终端

### 安装zsh
### 安装terminator
## neovim 安装

从vim 转移到neovim，个人觉得neovim用着可能更舒服一些，主要还是一些插件安装和用着舒服。

** 以上的三个配置为了方便移至，已经写了一个自动部署的脚本，详见[ubuntu_auto_config](http://github.com/suadminwen/ubuntu_auto_config) **

脚本完成之后，配置基本完成。
然而，没有那么智能，安装的过程中，总会有一些问题，所以还有根据具体问题进行修复。

## 截屏软件

截屏软件更新了一下，发现一个好用又强大的软件：火焰截图。

```
sudo apt install flameshot
```
如果apt 无法安装，我们也可以通过github下载安装包安装：[https://github.com/lupoDharkael/flameshot/releases](https://github.com/lupoDharkael/flameshot/releases)
选择ubuntu 18 版本的 `flameshot_0.6.0_bionic_x86_64.deb`下载，然后

```
sudo dpkg -i flameshot_0.6.0_bionic_x86_64.deb
```
安装完成之后，可以通过命令：`flameshot gui` 启动截图。

然后配置到快捷键便可以愉快的截图啦！

## Typora Markdown文档编辑器

Typora，到目前为止，认为的Ubuntu下最好的Markdown编辑器。

```
wget -qO - https://typora.io/linux/public-key.asc | sudo apt-key add -
sudo add-apt-repository 'deb https://typora.io/linux ./'
sudo apt-get update
sudo apt-get install typora
```

## guake 下拉终端

```
sudo apt install guake
```

安装完成之后，将guake添加到开机启动项中。

F12：显示/隐藏Guake的程序界面。

## electronic-wechat 

Mac OS X 和 Linux 下更好用的微信客户端. 更多功能, 更少bug. 使用Electron构建.

[github地址](https://github.com/kooritea/electronic-wechat)
