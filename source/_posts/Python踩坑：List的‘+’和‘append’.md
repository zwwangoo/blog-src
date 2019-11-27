---
title: Python踩坑：List的‘+’和‘append’
date: 2019-09-05
tags: [Python]
---


```python
In [2]: d = [1, 2, 1, 3, 2, 4]
   ...: print(id(d))
   ...: for i, v in enumerate(d):
   ...:     print(i, v)
   ...:     if v > 2:
   ...:         d = d[:i] + [1] + d[i:]
   ...:     print(id(d))
   ...:
```

```python

def func(data):
    print(data)
    print(id(data))
    # data.append('23')  # append 和 += 都不会创建新对象，
    # data += ['23']
    data = data + ['23']  # 这种写法，左边的data已经是一个新建的局部变量对象了,
    print(id(data))
    print(data)


if __name__ == '__main__':

    data = [12, 45, 89]
    func(data)
    print(data)
    print(id(data))

```
