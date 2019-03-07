---
title: Python有用的数据结构
date: 2019-03-06
tags: [python, 数据结构, 阅读笔记]
---

要改善应用程序的性能，最有效之一是使用更合适的算法和数据结构。Python 标准库提供了大量的现成的算法和数据结构，我们可以在直接使用它们。

## 列表和双端队列

Python 列表是有序的元素集合，在 Python 中使用大小可调整的数组来实现的。数组是一种基本数据结构，由一系列连续的内存单元组成，其中每个内存单元都包含指向一个 Python 对象的引用。

<!--more-->

在访问、修改和附加元素方面，列表表现的非常出色。在列表开头或中间添加或删除元素的操作，可能在效率方面存在问题。在列表开头插入或删除元素时，后续所有元素都需要移动一个位置，因此所需要的时间为O（N）。

在有些情况下，必须高效的执行集合开头和末尾插入或删除元素的操作，Python 通过`collections.deque`类提供一种具有这样特征的数据结构。deque指的是双端队列，在 Python 中，是以双向链表的方式实现的。所以`depue`提供了`pop`、`append`、`popleft`和`appendleft`，它们的运行时间都是O(1)，正是因为如此，付出的代价也是挺高的：**访问双端队列中间的元素所需的时间为O(N)**。例如下表所示：

|代码|N=1000(us)|N=2000(us)|N=3000|时间|
|-|-|-|-|-|
|duque[0]|0.37|0.41|0.45|O(1)|
|duque[N - 1]|0.37|0.42|0.45|O(1)|
|duque[int(N / 2)]|1.14|1.71|2.48|O(N)|

在列表中查找元素的索引可以通过list.index来完成的。为提高列表查找速度，一种简单的方法就是确保底层元素是有序的，并使用模块bisect来执行二分查找。

对于有序列表，函数bisect.bisect可以确定将元素插入到什么位置，同时可确保插入后列表依然是有序的。这个函数使用二分查找算法，运行时间为O(log(N))。


```python
import bisect
collection = [1, 2, 4, 5, 6]
bisect.bisect(collection, 3)
# 结果是 2
```

如果要插入的值已经包含在列表中，函数`bisect.bisect`将返回这个值后面的位置，因此可以使用`bisect.bisect_left`以下面的方式返回正确的索引：

```python
def index_bisect(a, x):
	i = bisect.bisect_left(a, x)
	if i != len(a) and a[i] == x:
		return i
	raise ValueError
```

该方法与`list.index`作用类似，但是运行时间几乎不受输入规模的影响，适合用来搜索**规模非常大而有序**的的集合。


## 字典

字典是以散列映射的方式实现的，在插入、删除和访问元素方面都是非常杰出的，这些操作的时间复杂度都是O(1)。

> 在Python3.5以及之前的版本中，字段是无序集合，但从3.6起字典能够保留元素的插入顺序。

下面演示几种使用字典高效的计算列表中独特元素的个数：

- 最基本的操作:

```python
def counter_dict(items):
	counter = {}
	for item in items:
		if item not in counter:
			counter[item] = 1
		else:
			counter[item] += 1
	return counter
```

- 使用`collections.defaultdict` 生成一个字典，并给每个新键自动指定一个默认值：

```python
from collections import defaultdict


def counter_defaultdict(items):
	counter = defaultdict(int)  # 每个新键都被自动指定零值
	for item in items:
		counter[item] += 1
	return counter

```

- 使用`collections.Counter`类：

```python
from collections import Counter
counter = Counter(items)
```

在速度方面，这些计数方式的时间复杂度都相同，但使用`Counter`实现的效率最高。

## 集合

集合是一个无序的元素集合，且其中的每个元素都必须是独一无二的。集合的主要用途是检查集合中是否包含特定的元素，集合的操作包含并集、差集和交集。

在Python中，集与字典一样，也是使用基于散列的算法实现的，因此其加法、删除、和成员资格测试(检查集合中是否包含特定的元素)等操作的时间复杂度都是O(1),即不受集合规模的影响。

集合中元素都是独一无二的，因此其一种显而易见的用途就是用于删除集合中重复的元素，为此只需将集合传递给构造函数`set`即可：

```python
x = list(range(1000)) + list(range(500))
x_unique = set(x)
# 集合x_unique将只包含x中不同的元素。
```

删除重复元素的时间复杂度为O(N)。


## 堆


堆是一种设计用于快速查找并提取集合中最大值或最小值的数据结构，其典型用途就是优先级处理一系列任务。堆的元素插入和最大值提取操作的时间复杂度都为O(log(N))。

在Python中，堆是通过堆列表执行模块heapq中的函数来创建的。


```
import heapq
collection = [10, 3, 3, 4, 4, 5, 6]
heapq.heapify(collection)
```

可使用`heapq.heappush`和`heapq.heappop`进行插入和提取操作。`heapq.heappop`提取集合中最小值：

```
heapq.heappop(collection)  # 返回3
```

同理要压入整数1可使用heapq.heappush:

```python
heapq.heapush(collection, 1)
```

## 字典树

字典树也成前缀树，这种数据结构可能不那么流行，但是很有用，在列表查找与前缀匹配的字符串方面，速度是极快的，因此非常适合用来实现输入时查找和自动补全功能。

Python标准库中不提供字典树的实现，可以通过PyPI找到很多高效的实现。推荐一个单文件的：`patricia-trie`。


参考：

- 《Python高性能(第2版)》第二章
