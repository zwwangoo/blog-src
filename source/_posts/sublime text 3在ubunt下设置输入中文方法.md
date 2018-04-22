---
title: sublime text 3在ubunt下设置输入中文方法
date: 2017-05-31
tags: [Linux, sublime]
---

## 将以下代码保存到`sublime_imfix.c`(位于`~`目录)

```
#include <gtk/gtkimcontext.h>
void gtk_im_context_set_client_window (GtkIMContext *context,
         GdkWindow    *window)
{
 GtkIMContextClass *klass;
 g_return_if_fail (GTK_IS_IM_CONTEXT (context));
 klass = GTK_IM_CONTEXT_GET_CLASS (context);
 if (klass->set_client_window)
   klass->set_client_window (context, window);
 g_object_set_data(G_OBJECT(context),"window",window);
 if(!GDK_IS_WINDOW (window))
   return;
 int width = gdk_window_get_width(window);
 int height = gdk_window_get_height(window);
 if(width != 0 && height !=0)
   gtk_im_context_focus_in(context);
}
```

<!--more-->

## 安装编译环境

```
sudo apt-get install build-essential

sudo apt-get install libgtk2.0-dev
```

## 将上一步的代码编译成共享库`libsublime-imfix.so`，命令

```
cd ~

gcc -shared -o libsublime-imfix.so sublime_imfix.c  `pkg-config --libs --cflags gtk+-2.0` -fPIC
```

## 然后将`libsublime-imfix.so`拷贝到sublime_text所在文件夹

```
sudo mv libsublime-imfix.so /opt/sublime_text/
```

## 修改文件`/usr/bin/subl`的内容

```
sudo gedit /usr/bin/subl
```
将
```
#!/bin/sh

exec /opt/sublime_text/sublime_text "$@"
```

修改为

```
#!/bin/sh

LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text "$@"
```

## 此时可在命令行执行命令subl可启动sublime-text 可输入汉字，但是点击图标则不行

## 在启动器快捷启动

找到sublime启动图标的放置位置

```
sudo find -iname sublime*
```

## find的结果如下
```
./.local/share/icons/sublime_text.png

./.local/share/applications/sublime_text.desktop

./.config/sublime-text-3
```
说明sublime启动图标位置为`./.local/share/applications/sublime_text.desktop`

## 然后修改启动设置

    sudo vim ./.local/share/applications/sublime_text.desktop

将`Exec=/opt/sublime_text/sublime_text`修改为

`Exec=/usr/bin/subl`

## 这时，不管时在终端优雅的输入 subl 还是在启动器中优雅的点击 sublime 进入后都可以输入中文啦！

## [参考资料](http://log.fyscu.com/index.php/archives/55/)