---
title: Python缓存函数执行结果lru_cache的使用
date: 2023-12-19
tags: [Python]
---

一个强大而有用的工具是 `functools` 模块中的 `lru_cache` 装饰器。
`lru_cache` 是 "Least Recently Used（最近最少使用）" 的缩写，是 Python 中用于缓存函数结果的装饰器。它的作用是在函数调用时缓存参数和结果，以便在后续相同参数的调用中直接返回缓存的结果，而不重新执行函数体。

## 基本用法

`lru_cache` 的主要优势之一是避免在相同参数下的重复计算。

```
from functools import lru_cache

@lru_cache(maxsize=None)
def fibonacci(n):
    if n <= 1:
        return n
    else:
        return fibonacci(n-1) + fibonacci(n-2)

# 第一次调用，计算 fibonacci(10)
result1 = fibonacci(10)

# 第二次调用，直接从缓存返回结果
result2 = fibonacci(10)

print(result1, result2)  # 输出: 55 55
```

第一次调用 `fibonacci(10)` 会计算结果并将其缓存起来。第二次调用相同的参数时，直接从缓存返回结果，而不重新计算。

## 对象方法中的应用

`lru_cache` 不仅仅可以用于普通函数，还可以用于类的方法。如果方法的结果仅取决于其参数，并且不依赖于对象的状态，那么可以使用 `lru_cache` 提高性能。

```
from functools import lru_cache

class MyClass:
    def __init__(self, name):
        self.name = name

    @lru_cache(maxsize=None)
    def get_data(self, value):
        print(f"Loading data for {self.name} with value {value}")
        return value + 10

# 创建两个实例
obj1 = MyClass("Instance 1")
obj2 = MyClass("Instance 2")

# 对于 obj1，第一次调用 get_data 时会加载数据
print(obj1.get_data(5))  # 输出: Loading data for Instance 1 with value 5
# 对于 obj1，第二次调用相同的参数，直接从缓存返回
print(obj1.get_data(5))  # 输出: 15 (从缓存返回)

# 对于 obj2，第一次调用 get_data 时会加载数据
print(obj2.get_data(5))  # 输出: Loading data for Instance 2 with value 5
# 对于 obj2，第二次调用相同的参数，直接从缓存返回
print(obj2.get_data(5))  # 输出: 15 (从缓存返回)
```

注意的是 **如果使用 @staticmethod 装饰器装饰的方法，那么 lru_cache 缓存会是共享的，不同实例之间的缓存是相同的。这是因为静态方法没有实例作为第一个参数，它是在类级别上调用的，因此只有一个缓存。**

```
from functools import lru_cache

class MyClass:
    @staticmethod
    @lru_cache(maxsize=None)
    def static_method(value):
        print(f"Loading data for value {value}")
        return value + 10

# 对于静态方法，缓存是共享的
# 第一次调用 static_method 时会加载数据
print(MyClass.static_method(5))  # 输出: Loading data for value 5
# 第二次调用相同的参数，直接从缓存返回
print(MyClass.static_method(5))  # 输出: 15 (从缓存返回)
```

对于静态方法而言，它是在类级别上调用的，而不是在实例级别上。即使实例化后调用，也会共享相同的缓存：

```
my_class1 = MyClass()
print(my_class1.static_method(5))  # 第一次调用，输出: Loading data for value 5
print(my_class1.static_method(5))  # 第二次调用，输出: 15 (从缓存返回)

my_class2 = MyClass()
print(my_class2.static_method(5))  # 通过另一个实例调用，输出: 15 (从缓存返回)
```
