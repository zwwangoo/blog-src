---
title:  Python3的10个令人敬畏的功能
date: 2018-05-03
tags: [python]
---

这是一篇翻译的文章，目的是为了更好的学习python，使用python3的强大特性。原文[点击这里](https://www.asmeurer.com/python3-presentation/slides.html#1)

## 你拒绝升级到Python 3，因此无法使用Python的10个令人敬畏的功能

### 特性一 高级拆包

- 你可以这么做：

```python
a, b = range(2)
print(a)  # 0
print(b)  # 1
```

<!--more-->

- 但是现在，你可以这么做了：

```python
a, b, *rest = range(10)
print(a)  # 0
print(b)  # 1
print(rest)  # [2, 3, 4, 5, 6, 7, 8, 9]
```

- `*rest` 能够放到任何地方

```python
a, *rest, b = range(10)
print(a)  # 0
print(b)  # 9
print(rest)  # [1, 2, 3, 4, 5, 6, 7, 8]
```

```python
*rest, a = range(10)

print(rest)  # [0, 1, 2, 3, 4, 5, 6, 7, 8]
print(a)  # 9
```

- 得到文件的第一行和最后一行

```python
with open("using_python_to_profit") as f:
    first, *_, last = f.readlines()
```

### 特性二 限定关键字参数

```python
def func(a, b, *args, option=True):
    ...
```

- `option` 在`*args` 的后面 
- 访问它的唯一方法是显式调用`func(a, b, option=True)`
- 如果你不想传入 `*args`参数，你可以传入 `*`