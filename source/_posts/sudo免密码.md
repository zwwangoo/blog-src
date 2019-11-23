---
title: sudo免密码
date: 2018-11-13
tags: [Ubuntu]
---

 `sudo  vi /etc/sudoers`
 
将
```
%sudo   ALL=(ALL:ALL)  ALL
```
改为
```
%sudo   ALL=(ALL:ALL) NOPASSWD: ALL
```

保存后退出

之后输入 `sudo` 就可以不用输入密码。 

至于安全问题，对于一般个人用户，我觉得这样也可以的。

如果改崩了，可以使用 `pkexec visudo` 输入密码后修改对应出错的地方，保存文件即可解决此问题
