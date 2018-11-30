---
title: Ubuntu18 给自定义应用程序快捷方式
date: 2018-11-12
tags: [Ubuntu]
---

## Ubuntu 创建快捷启动应用程序

在 $HOME/.local/share/applications 目录下，新建文件： youapp_name.desktop，然后在其中填入以下内容：

```
[Desktop Entry]

Encoding=UTF-8

Name=Navicat

Exec=/opt/navicat121_mysql_en_x64/start_navicat
Icon=/opt/navicat121_mysql_en_x64/icon.png

Terminal=false

Type=Application

Categories=Development;
```

其中  `Exec` 对应的是启动命令，如果是自己写的脚本，那就在以 `Exec=sh` 开头。
`Icon` 对应的是图标

## Ubuntu18 如何将快捷方式锁定在启动器上

长按图标，然后选择添加到收藏夹即可
