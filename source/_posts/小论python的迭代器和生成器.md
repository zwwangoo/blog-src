---
title: 小论python的迭代器和生成器
date: 2017-09-09
tags: [python]
---

> 《python核心编程》的第一部分读完之后，开始总结python的基础结构图，觉得生成器和迭代器的关系错综复杂，于是乎，没事呀，就研究研究吧。

## 迭代器

### 内建函数iter()

list,tuple,dict,set,str等集合数据类型不是迭代器对象Iterator但它们是可迭代对象Iterable，可以使用iter()方法将Iterable变成Iterator对象

<!--more-->

### 自定义迭代器

自定义迭代器必须实现`__iter__()`和`next()`方法

```python

class d:
    def __init__(self, data):
        self.date = data
        self.index = len(data)
        
    def __iter__(self):
        return self
        
    def next(self):
        if self.index == 0:
            raise StopIteration
        self.index = self.index - 1
        return self.date[self.index]
    

dh = d([1, 5])
print type(dh)  # <type 'instance'>

for x in dh:
    print x

``` 

## 生成器

生成器是一种特殊的迭代器，内部支持了生成器协议，不需要明确定义__iter__()和next()方法

```python
def b():
    yield 12
    yield 13
bf = b()
for i in bf:
    print i
```

生成器中还有两个很重要的方法：send()和close()。

### send(value):

从前面了解到，next()方法可以恢复生成器状态并继续执行，其实send()是除next()外另一个恢复生成器的方法。

Python 2.5中，yield语句变成了yield表达式，也就是说yield可以有一个值，而这个值就是send()方法的参数，所以send(None)和next()是等效的。同样，next()和send()的返回值都是yield语句处的参数（yielded value）

关于send()方法需要注意的是：调用send传入非None值前，生成器必须处于挂起状态，否则将抛出异常。也就是说，第一次调用时，要使用next()语句或send(None)，因为没有yield语句来接收这个值。

### close():
这个方法用于关闭生成器，对关闭的生成器后再次调用next或send将抛出StopIteration异常。

## 生成器和迭代器主要有以下几点区别：

- 迭代器是一个对象，生成器是一个函数。
- 迭代器包含`__iter__()` 和`next()`方法`__iter__()`返回迭代器本身(self)，next()返回next()方法返回容器的下一个元素；生成器有生成器函数生成，但是不用return返回，而是yield一次返回一个结果，且“暂停”代码的执行。
- 生成器是一种特殊的迭代器，可使用next()方法。

```python
def a():
    yield 1
    yield 2
af = a()
af.next()  # 1
af.next()  # 2
```

