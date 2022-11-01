---
title: Pycharm在使用ideavim时，进入normal模式时，输入法自动切换到英文状态
date: 2022-10-11
tags: [Python, VIM, MACOS]
---

如果您已经是IdeaVim的用户,那么直接在插件中心搜索IdeaVimExtension进行安装.或者到IdeaVimExtension插件主页进行下载安装.IdeaVimExtension是依赖IdeaVim的,需要事先安装IdeaVim

1. 确保你的操作系统已经开启了英文输入法
    * Windows需要开启en_US输入法
    * macOS需要开启ABC或en_US输入法
    * Linux 不支持
2. 安装重启IDEA后,打开任意代码编辑器在normal模式下输入如下两个命令来激活IdeaVimExtension插件
    * `:set keep-english-in-normal `: 在normal模式保持英文状态
    * `:set keep-english-in-normal-and-restore-in-insert` : 在normal模式保持英文状态,并在回到insert时恢复输入法到原来的状态.例如,编写一段中文注释,用中文输入法写了一段文字,进入normal模式移动光标到下一行,再回到插入模式继续使用中文编辑.
3. 上面两个命令在每次IDEA重启后都需要重新输入,也可以通过在用户目录下添加` .ideavimrc`文件,将命令添加到该文件中,这样在IDEA重启时可以自动执行该文件中的指令.