---
title: Python双向队列collections.deque
date: 2020-01-05
tags: [Python, 数据结构]
---

deque队列是由栈或者queue队列生成的，支持线程安全（也就是说你可以**同时从deque集合的左边和右边进行操作而不会有影响**），从两端都可以高效的添加(append)和弹出(pop)元素，两个方向的大概开销都是 O(1) 复杂度。

在创建时，可以指定最大长度`collections.deque(maxlen=x)` ，如果 *maxlen* 没有指定或者是 `None` ，deques 可以增长到任意长度。否则，deque就限定到指定最大长度，当deque满了，有新项加入时，同样数量的项就从另一端弹出。

```python
In [1]: import collections
In [2]: d = collections.deque(maxlen=3)
In [3]: d.append(1)
In [4]: d.append(2)
In [5]: d.append(3)
In [6]: d
Out[6]: deque([1, 2, 3])
In [7]: d.pop()
Out[7]: 3
In [8]: d
Out[8]: deque([1, 2])
```

双向队列(deque)对象支持以下方法:

- `append(x)` 添加 *x* 到右端
- `appendleft(x)` 添加x到左端
- `pop()` 移去并且返回最右侧的元素，如果没有元素的话，就抛出 `IndexError` 索引错误。
- `popleft()` 移去并且返回最左侧的元素，如果没有元素的话，就抛出 `IndexError` 索引错误。
- `clear()` 移除所有元素，使其长度为0
- `remove(x)` 移去找到的第一个 *value*。 如果没有，抛出`ValueError`
- `copy()` 创建一份浅拷贝
- `count()` 计算等于 *x* 的元素个数
- `extend(iterable)` 将iterable中的元素添加到deque的右侧
- `extendleft(iterable)` 将iterable中的元素添加到deque的左侧，在结果中iterable中的顺序将被反过来添加。
- `index(x[, start[, stop]])` 返回第一个匹配的第 *x* 个元素（从 *start* 开始计算，在 *stop* 之前），如果没找到的话，抛出`ValueError`
- `insert(i, x)` 在i位置插入x，如果插入会导致deque超出长度 *maxlen* 的话，就抛出 `IndexError` 索引错误
- `reverse()` 将deque逆序排列。返回 `None` 。
- `rotate(n=1)` 向右循环移动 *n* 步。 如果 *n* 是负数，就向左循环。如果deque不是空的，向右循环移动一步就等价于 `d.appendleft(d.pop())` ， 向左循环一步就等价于 `d.append(d.popleft())` 。

除了以上的方法，deque还支持迭代、成员测试、下标引用(d[-1])等等：

```
In [1]: import collections
In [2]: d = collections.deque(maxlen=3)
In [3]: d.append(1)
In [4]: d.append(2)
In [5]: d.append(3)
In [6]: 2 in d
Out[6]: True
In [7]: d[-1]
Out[7]: 3
In [8]: d[1]
Out[8]: 2
In [9]: len(d)
Out[9]: 3
In [10]: if d:  # 判断队列d是否为空
    ...:     print(d.pop())
3
In [12]: d
Out[12]: deque([1, 2])
In [13]: d.extend([4, 5, 6])
In [14]: d
Out[14]: deque([4, 5, 6])

In [15]: d = collections.deque('abcdef')  # 重新赋值
In [16]: d
Out[16]: deque(['a', 'b', 'c', 'd', 'e', 'f'])
```

参考：

- [collections --- 容器数据类型](https://docs.python.org/zh-cn/3.7/library/collections.html#deque-objects)