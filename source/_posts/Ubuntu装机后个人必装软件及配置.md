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
```

安装完成之后，命令行启动

`/usr/bin/google-chrome-stable`

启动完成之后锁定到启动器即可。

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

## 搜狗输入法

## 美化终端

### oh-my-zsh的安装及个人配置

安装oh-my-zsh之前，需要保证zsh已经被安装

#### 安装zsh

- 首先需要安装zsh

`sudo apt-get install zsh`

- 确认是否安装成功

`zsh --version` 

- 设置zsh为默认shell

`sudo chsh -s $(which zsh)`

设置完成之后，需要注销重新登录

#### 安装oh-my-zsh

- 下载并安装：

```
sudo apt install curl
curl -L https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh | sh 
```

- 配置：

```
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
chsh -s /bin/zsh
```

- 个人设置

编辑`~/.zshrc`,更改
```
ZSH_THEME='terminalparty'
```
添加
```
export DEFAULT_USER='username'
```

以下是关于pyenv的配置，如果安装pyenv则需要在该文件中添加如下语句：
```
export PATH="/home/ouou/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

### 安装 terminator

记住常用的快捷键

安装terminator

`sudo apt-get install terminator`

对terminator进行配置：`~/.config/terminator/config`

> 如果报错，Unable to open ~/.config/terminator/config ，解决方法：打开terminator终端，然后右击终端的黑色背景，选择preference->layouts->add，关闭该窗口即可找到config文件。

> 参考[这里](https://www.aliyun.com/jiaocheng/119215.html)

```
[global_config]
  borderless = True
  enabled_plugins = CustomCommandsMenu, LaunchpadCodeURLHandler, APTURLHandler, LaunchpadBugURLHandler
  handle_size = -3
  inactive_color_offset = 1.0
  suppress_multiple_term_dialog = True
  title_font = Sans 14
  title_transmit_bg_color = "#3e3838"
  title_transmit_fg_color = "#000000"
  window_state = fullscreen
[keybindings]
  close_term = <Super>c
  cycle_next = <Alt>l
  cycle_prev = <Alt>h
  go_down = None
  go_left = None
  go_right = None
  go_up = None
  help = None
  layout_launcher = None
  split_horiz = <Alt>o
  split_vert = <Alt>e
[layouts]
  [[default]]
    [[[child1]]]
      parent = window0
      type = Terminal
    [[[window0]]]
      parent = ""
      type = Window
[plugins]
[profiles]
  [[default]]
    background_darkness = 0.9
    background_image = None
    background_type = transparent
    cursor_color = "#ffffff"
    custom_command = tmux
    font = 文泉驿等宽微米黑 12
    foreground_color = "#ffffff"
    login_shell = True
    palette = "#000000:#5a8e1c:#2d5f5f:#cdcd00:#1e90ff:#cd00cd:#00cdcd:#e5e5e5:#4c4c4c:#868e09:#00ff00:#ffff00:#4682b4:#ff00ff:#00ffff:#ffffff"
    show_titlebar = False
    use_system_font = False
```

## vim的配置

## 截屏软件

安装`deepin-screenshot`
先安装一些必须安装的库

```
sudo apt install python-deepin-utils python-imaging python-scipy python-wnck python-xdg
```

下载安装包进行安装：

```
wget http://packages.linuxdeepin.com/deepin/pool/main/d/deepin-utils/python-deepin-utils_0.0.1-1%2bgit20130502093354ubuntu2_amd64.deb \
         http://packages.linuxdeepin.com/deepin/pool/main/d/deepin-gsettings/deepin-gsettings_0.1%2bgit20130318115600ubuntu1_amd64.deb \
         http://packages.linuxdeepin.com/deepin/pool/main/d/deepin-ui/deepin-ui_1%2bgit20130618094833~90b6485f6cprecise2_all.deb \
         http://packages.linuxdeepin.com/deepin/pool/main/d/deepin-screenshot/deepin-screenshot_2.0%2bgit20130618155753~8c1ba38453precise_all.deb

sudo dpkg -i python-deepin-utils_0.0.1-1+git20130502093354ubuntu2_amd64.deb 
sudo dpkg -i deepin-gsettings_0.1+git20130318115600ubuntu1_amd64.deb
sudo dpkg -i deepin-ui_1+git20130618094833~90b6485f6cprecise2_all.deb
sudo dpkg -i deepin-screenshot_2.0+git20130618155753~8c1ba38453precise_all.deb
```
安装的过程中，如果出现**依赖关系问题 - 仍未被配置**等问题，执行`sudo apt -f install `之后重新执行安装命令即可。

安装完成之后，在终端输入命令`deepin-screenshot`就会出现截屏的界面，然后自定义快捷键`ctrl+alt+a`启动截屏

## 网易云音乐

官网更新的1.1.0版本安装完成之后，之后超级管理员能够启动，普通管理员无权限，所以要安装1.0.0版本的。
下载安装包：

执行命令 

`sudo dpkg -i `
