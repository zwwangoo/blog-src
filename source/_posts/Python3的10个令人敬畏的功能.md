---
title:  Python3的10个令人敬畏的功能
date: 2018-05-03
tags: [python]
---

这是一篇翻译的文章，目的是为了更好的学习python，使用python3的强大特性。原文[点击这里](https://www.asmeurer.com/python3-presentation/slides.html#1)

## 因为你拒绝升级到Python 3，因此无法使用Python的10个令人敬畏的功能

### 特性1 高级拆包

- 你可以这么做：

```python
>>> a, b = range(2)
>>> a
0
>>> b
1
```

<!--more-->

- 但是现在，你可以这么做了：

```python
>>> a, b, *rest = range(10)
>>> a
0
>>> b
1
>>> rest
[2, 3, 4, 5, 6, 7, 8, 9]
```

- `*rest` 能够放到任何地方

```python
>>> a, *rest, b = range(10)
>>> a
0
>>> rest
[1, 2, 3, 4, 5, 6, 7, 8]
>>> b
9
```

```python
>>> *rest, a = range(10)

>>> rest
[0, 1, 2, 3, 4, 5, 6, 7, 8]
>>> a
9
```

- 得到文件的第一行和最后一行

```python
with open("using_python_to_profit") as f:
    first, *_, last = f.readlines()
```

### 特性二 限定关键字参数