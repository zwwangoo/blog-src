---
title: Macos使用CapLock切换搜狗输入法
date: 2022-07-11
tags: [MACOS]
---


1、依赖软件：

- Karabiner
- 搜狗输入法


2、实现步骤

(1)停用搜狗输入法的英文输入模式（默认中文，不设置中英文切换快捷键）
(2)用 Karabiner 改键 CapsLock 为 f18
(3)系统偏好设置->键盘->快捷键->输入法 配置：

- 选择上一个输入法 F18
- 选择“输入法”菜单中的下一个输入法 ⌥F18  (同时按下 Option + CapsLock)


3、进阶自动切换输入法

- 依赖软件 KeyboardHolder
