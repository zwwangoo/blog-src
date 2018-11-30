---
title: sudo免密码
date: 2018-11-13
tags: [Ubuntu]
---

1) 首先需要切换到root, su -

2) 然后 vi /etc/sudoers
在 `root ALL=(ALL) ALL`的下一行，输入

```
your_user_name ALL=(ALL) ALL
```
保存后退出

这样就把自己加入了sudo组，可以使用sudo命令了。

3) 默认5分钟后刚才输入的sodo密码过期，下次sudo需要重新输入密码，如果觉得在sudo的时候输入密码麻烦，把刚才的输入换成如下内容即可：

```
your_user_name ALL=(ALL) NOPASSWD: ALL
```

至于安全问题，对于一般个人用户，我觉得这样也可以的。

**注：** 有的时候你的将用户设了nopasswd，但是不起作用，原因是被后面的group的设置覆盖了，需要把group的设置也改为nopasswd。

```
%admin ALL=(ALL) NOPASSWD: ALL
```
