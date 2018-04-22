---
title: Linux踢出其他正在SSH登陆用户
date: 2016-12-23
tags: [Linux]
---

## 查看系统在线用户

```
[root@testdb ~]# w

14:30:26 up 38 days, 21:22, 3 users, load average: 0.00, 0.01, 0.05
USER TTY FROM LOGIN@ IDLE JCPU PCPU WHAT
root pts/0 162.16.16.155 14:30 0.00s 0.07s 0.05s w
root pts/1 162.16.16.155 14:30 12.00s 0.01s 0.01s -bash
```

## 查看哪个属于此时自己的终端（我开了两个连接）

```
[root@testdb ~]# who am i

root pts/0 2013-12-31 14:30 (162.16.16.155)
```

## pkill掉自己不适用的终端

    [root@testdb ~]# pkill -kill -t pts/1

## 查看当前终端情况

```
[root@testdb ~]# w
14:31:04 up 38 days, 21:23, 2 users, load average: 0.00, 0.01, 0.05
USER TTY FROM LOGIN@ IDLE JCPU PCPU WHAT
root pts/0 162.16.16.155 14:30 0.00s 0.04s 0.01s w
[root@testdb ~]#
```
注意：

如果最后查看还是没有干掉，建议加上-9 强制杀死。

    [root@testdb ~]# pkill -9 -t pts/1


