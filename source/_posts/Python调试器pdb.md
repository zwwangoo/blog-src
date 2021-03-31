---
title: Python调试器pdb
date: 2021-03-31
tags: [Python, 调试]
---

- [10分钟教程掌握Python调试器pdb](https://zhuanlan.zhihu.com/p/37294138)


## 使用方式

- `$ python -m pdb test.py` **非侵入式方法**，不用额外修改源代码，在命令行下直接运行就能调试

- `import pdb;pdb.set_trace()`**侵入式方法**，需要在被调试的代码中添加一行代码然后再正常运行代码

  当你在命令行看到下面这个提示符时，说明已经正确打开了`pdb`

  ```bash
  (Pdb)
  ```

## 常用命令

### 查看源码

- `l` 查看当前位置前后11行源代码（多次会翻页），当前位置在代码中会用`-->`这个符号标出来
- `ll` 查看当前函数或框架的所有源代码

### 添加断点

- `b` 查看断点设置
- `b lineno` 断点添加到哪一行
- `b filename:lineno` 断点添加到哪个文件的哪一行
- `b functionname` 在函数执行的第一行设置断点

### 添加临时断点，执行一次后时自动删除

- `tbreak` 查看临时断点
- `tbreak lineno`
- `tbreak filename:lineno`
- `tbreak functionname`

### 清除断点

- `cl` 清除所有断点
- `cl filename:lineno`
- `cl bpnumber [bpnumber ...]` 清除指定序号断点

### 打印变量

- `p expression`  也可以是表达式
- `pp expression` 打印好看的

### 逐行调试

- `s` 执行下一行可以进入函数体
- `n` 执行下一行不会进入函数体
- `r` 执行下一行，如果在函数中，会直接执行到函数返回处

### 非逐行调试

- `c` 持续执行下去，直到遇到一个断点
- `unt lineno` 持续执行直到运行到指定行（或遇到断点）
- `j lineno` 直接跳转到指定行（注意，被跳过的代码不执行）

### 其他

- `a` 在函数中时打印函数的参数和参数的值
- `! expression` 在pdb中执行语句，注意语句中不能出现空格
- `whatis expression` 打印表达式的类型，常用来打印变量值
- `interact` 启动交互式解释器
- `q` 退出pdb