---
title: python使用的一些小技巧和面试中遇到的问题整理
date: 2018-3-12
tags: [python]
---

> 2018年，新的开始，所以我用Python3.x啦，除了原有项目的维护外，全面升级到python3，人生苦短，就用python
## 6月26日

值得参考的python[点击这里](https://taizilongxu.gitbooks.io/stackoverflow-about-python/content/1/README.html)

### 字符串格式化：% 和.format

.format 在很多方便看起来更便利，对于 % 最烦人的是它无法同时传递一个变量和元祖，例如以下：

```
"hi there %s" % name 
```
但是，如果name恰好是(1, 2, 3）,它会抛出一个TypeError异常.为了保证它总是正确的,你必须这样做:

```
"hi there %s" % (name,)   # 提供一个单元素的数组而不是一个参数
```
但是有点丑..format就没有这些问题.你给的第二个问题也是这样,.format好看多了.

### 字典推导式

```
d = {key: value for (key, value) in iterable}
```

## 3月12日

### windows系统中不同版本python存在的应用问题

- 在windows系统中，同时存在python3.x和python2.x：

命令：`py -2` 启动python2.7的交互解释器；`py -3` 启动python3.x的交互解释器

- 如何使用不同版本的pip：

命令：`py -2 -m pip` 使用2.x的pip；`py -3 -m pip` 使用3.x的pip

- pycharm使用不同版本的python

<!--more-->


## 2月28日

补充：

### `__new__`和`__init__`的区别

- `__init__`为初始化方法，`__new__`方法是真正的构造函数。
- `__new__`是实例创建之前被调用，它的任务是创建并返回该实例，是静态方法
- `__init__`是实例创建之后被调用的，然后设置对象属性的一些初始值。

总结：`__new__`方法在`__init__`方法之前被调用，并且`__new__`方法的返回值将传递给`__init__`方法作为第一个参数，最后`__init__`给这个实例设置一些参数。


### python的自省

自省就是面向对象的语言所写的程序在运行时，所能知道对象的类型。简单一句话就是运行时能够获得对象的类型。比如：`type()`、`dir()`、`getattr()`、`hasattr()`、`isinstance()`

## 1月26日

以下是根据面试的[笔试题](https://suadminwen.github.io/2018/02/26/%E5%B0%8F%E4%BD%99%E6%95%99%E8%82%B2%E9%9D%A2%E7%BB%8F%E5%8F%8A%E6%95%B4%E7%90%86/)整理的部分相关知识：

### 字符串前添加的`r`、`u`、`b`的含义。

- `u/U`: 表示unicode字符串。
- `r/R`: 非转义的原始字符串。以r开头的字符，常用于正则表达式，对应着re模块。
- `b`: bytes


**注意：**

python3里默认的`str`是(py2.x里的)`unicode`, `bytes`是(py2.x)的`str`, `b""`前缀代表的就是`bytes`
python2里, `b`前缀没什么具体意义。

### 生成器和迭代器[参看这里](https://suadminwen.github.io/2017/09/09/%E5%B0%8F%E8%AE%BApython%E7%9A%84%E8%BF%AD%E4%BB%A3%E5%99%A8%E5%92%8C%E7%94%9F%E6%88%90%E5%99%A8/)

### 切片和切片步长的问题。

对于具有序列结构的数据来说，切片操作的方法是：consequence[start_index: end_index: step]

- start_index：表示是第一个元素对象，正索引位置默认为0；负索引位置默认为 -len(consequence)
- end_index：表示是最后一个元素对象，正索引位置默认为 len(consequence)－1；负索引位置默认为 -1。
- step：表示取值的步长，默认为1，步长值不能为0。

利用步长对序列进行倒序取值：
```
a=[1,2,3,4,5,6,7]
print(a[::-1])  # 这里不会改变a的结构
a.reverse()  # 这里a的结构被改变了
print(a)
```

### 垃圾回收机制和循环引用问题

Python的GC模块主要运用了 **“引用计数”（reference counting）** 来跟踪和回收垃圾。在引用计数的基础上，还可以通过 **“标记-清除”（mark and sweep）** 解决容器对象可能产生的循环引用的问题。通过 **“分代回收”（generation collection）** 以空间换取时间来进一步提高垃圾回收的效率。[详细参考这里](http://python.jobbole.com/82061/)

#### 引用计数

PyObject是每个对象必有的内容，其中ob_refcnt就是做为引用计数。当一个对象有新的引用时，它的ob_refcnt就会增加，当引用它的对象被删除，它的ob_refcnt就会减少.引用计数为0时，该对象生命就结束了。

#### 标记-清除机制

基本思路是先按需分配，等到没有空闲内存的时候从寄存器和程序栈上的引用出发，遍历以对象为节点、以引用为边构成的图，把所有可以访问到的对象打上标记，然后清扫一遍内存空间，把所有没标记的对象释放。

#### 分代技术

分代回收的整体思想是：将系统中的所有内存块根据其存活时间划分为不同的集合，每个集合就成为一个“代”，垃圾收集频率随着“代”的存活时间的增大而减小，存活时间通常利用经过几次垃圾回收来度量。

Python默认定义了三代对象集合，索引数越大，对象存活时间越长。

举例： 当某些内存块M经过了3次垃圾收集的清洗之后还存活时，我们就将内存块M划到一个集合A中去，而新分配的内存都划分到集合B中去。当垃圾收集开始工作时，大多数情况都只对集合B进行垃圾回收，而对集合A进行垃圾回收要隔相当长一段时间后才进行，这就使得垃圾收集机制需要处理的内存少了，效率自然就提高了。在这个过程中，集合B中的某些内存块由于存活时间长而会被转移到集合A中，当然，集合A中实际上也存在一些垃圾，这些垃圾的回收会因为这种分代的机制而被延迟。

### `staticmethod()`与`classmethod()`的区别

一般来说，要使用某个类的方法，需要先实例化一个对象再调用方法。而使用`@staticmethod`或`@classmethod`，就可以不需要实例化，直接类名.方法名()来调用。

- `@staticmethod`不需要表示自身对象的`self`和自身类的`cls`参数，就跟使用函数一样。
- `@classmethod也`不需要`self`参数，但第一个参数需要是表示自身类的`cls`参数。(`cls`:它表示这个类本身。)

如果在`@staticmethod`中要调用到这个类的一些属性方法，只能直接类名.属性名或类名.方法名。而`@classmethod`因为持有`cls`参数，可以来调用类的属性，类的方法，实例化对象等，避免硬编码。

```python
class A(object):  
    bar = 1  
    def foo(self):  
        print 'foo'  

    @staticmethod  
    def static_foo():  
        print 'static_foo'  
        print A.bar  

    @classmethod  
    def class_foo(cls):  
        print 'class_foo'  
        print cls.bar  
        cls().foo()    # 这里需要注意的是cls()
###执行  
A.static_foo()  
A.class_foo()  
```

### python中的单例模式

**单例模式（Singleton Pattern）** 是一种常用的软件设计模式，该模式的主要目的是确保某一个类只有一个实例存在。[]参考](https://funhacks.net/2017/01/17/singleton/)

#### 使用模块

**Python 的模块就是天然的单例模式，因为模块在第一次导入时，会生成 .pyc 文件，当第二次导入时，就会直接加载 .pyc 文件，而不会再次执行模块代码。** 因此，我们只需把相关的函数和数据定义在一个模块中，就可以获得一个单例对象了。如果我们真的想要一个单例类，可以考虑这样做：

```python
# mysingleton.py
class My_Singleton(object):
    def foo(self):
        pass

my_singleton = My_Singleton()


# test.py
from mysingleton import my_singleton

my_singleton.foo()
```

#### 使用 `__new__`

使用 `__new__` 来控制实例的创建过程

```python
class Singleton(object):
    _instance = None
    def __new__(cls, *args, **kw):
        if not cls._instance:
            cls._instance = super(Singleton, cls).__new__(cls, *args, **kw)  
        return cls._instance  

class MyClass(Singleton):  
    a = 1

```

#### 使用装饰器

装饰器（decorator）可以动态地修改一个类或函数的功能。

```python
def singleton(cls, *args, **kw):
    instance = {}
    def get_singleton():
        if cls not in instance:
            instance[cls] = cls(*args, **kw)
        return instance[cls]
    return get_singleton


@singleton
class TT(object):
    def __init__(self):
        self.num = 0
    def add(self):
        self.num = 100


if __name__ == '__main__':
    a = TT()
    print(a.num)  # 0
    a.add()
    b = TT()
    print(b.num)  # 100
```

虽然进行了两次实例化，但仍为同一个实例

#### 使用 元类(metaclass)

后面会有专门的一部分来整理元类的问题


### 浅拷贝和深拷贝的区别

这篇[文章](http://songlee24.github.io/2014/08/15/python-FAQ-02/)很好理解!

#### 浅拷贝

浅拷贝（shallow copy）指创建一个新的对象，其内容是原对象中元素的引用。（拷贝父对象，不拷贝子对象）

常见的浅拷贝有：**切片操作**、**工厂函数**、**对象的copy()** 方法、**copy模块中的copy函数**。

```python
a = [1, 2, 3]
b = list(a)  # 这里是浅拷贝
c = a  # 这里为赋值
print(id(a), id(b), id(c))  # 2130788487752 2130788487880 2130788487752 a与c指向相同
a.append(4)
print(a, b, c)  # [1, 2, 3, 4] [1, 2, 3] [1, 2, 3]
```

#### 深拷贝

深拷贝（deep copy）是指创建一个新的对象，然后递归的拷贝父对象和子对象。深拷贝出来的对象与原对象没有任何关联。

深拷贝只有一种方式：copy模块中的deepcopy函数。

```python
import copy

a = [1, 2, 3]
b = copy.deepcopy(a)
print(id(a), id(b))
```

对于不可变对象，当需要一个新的对象时，python可能会返回已经存在的某个类型和值都一致的对象的引用。而且这种机制并不会影响 a 和 b 的相互独立性，因为当两个元素指向同一个不可变对象时，对其中一个赋值不会影响另外一个。


#### 用一个包含可变对象的列表来确切地展示“浅拷贝”与“深拷贝”的区别：

```python
import copy
a = [[1, 2], [5, 6], [8, 9]]
b = copy.copy(a)
c = copy.deepcopy(a)
print(id(a), id(b), id(c))

for x, y in zip(a, b):
    print(id(x), id(y))  # 输出的id相同

for x, y in zip(a, c):
    print(id(x), id(y))  # 输出的id不同

```

#### 总结

- 1、赋值：简单地拷贝对象的引用，两个对象的id相同。
- 2、浅拷贝：创建一个新的组合对象，这个新对象与原对象共享内存中的子对象。
- 3、深拷贝：创建一个新的组合对象，同时递归地拷贝所有子对象，新的组合对象与原对象没有任何关联。虽然实际上会共享不可变的子对象，但不影响它们的相互独立性。
- 4、浅拷贝和深拷贝的不同仅仅是对组合对象来说，所谓的组合对象就是包含了其它对象的对象，如列表，类实例。而对于数字、字符串以及其它“原子”类型，没有拷贝一说，产生的都是原对象的引用。

### 继承和实现的问题

题目：类A、B、C、D、E，C继承于A，D继承于B，E继承于C、D，C中和B中都有demo()方法，问，调用E中的demo()，C和B中谁的demo()会被调用？

```python
class A(object):
    def __init__(self):
        print('init A...')

    def getoo(self):
        print("getoo")


class B(object):
    def __init__(self):
        print('init B...')

    def demo(self):
        print("B")


class C(A):
    def __init__(self):
        super(C, self).__init__()
        print('init C...')

    def demo(self):
        print("C")


class D(B):
    def __init__(self):
        super(D, self).__init__()
        print('init D...')


class E(D,C):  # 继承的先后顺序不一样，则输出结果是不一样的
    def __init__(self):
        super(E, self).__init__()
        print('init E...')


if __name__ == '__main__':
    e = E()
    e.demo()
    e.getoo()
'''
输出：
init B...
init D...
init E...
B
getoo
'''
```

### 元类的问题（metaclass）

元类就是用来创建这些类（对象）的，元类就是类的类。函数`type`实际上是一个元类。`type`就是Python在背后用来创建所有类的元类。`str`是用来创建字符串对象的类，而`int`是用来创建整数对象的类。`type`就是创建类对象的类。

```python
a = 12
print(a.__class__)  # <class 'int'>
print(a.__class__.__class__)  # <class 'type'>
```

你可以在写一个类的时候为其添加`__metaclass__`属性,定义了`__metaclass__`就定义了这个类的元类。


---
## 1月25之前

### 1. 需要在循环中使用索引，可以使用 `enumerate()` ：

```python
for index, value in enumerate(alist):
    print(index, value)
```

### 2.  需要同时迭代两个循环，用同一个索引来获取两个值，可以用 `zip()`

```python
for word, number in zip(words, numbers):
    print word, number  # 其中一个迭代结束，就结束迭代了
```

### 3. python3中xrange()已经不存在了，range()已经实现了xrange的功能。range的一个重要用法是当你真正想要生成一个数字序列而不是用来生成索引。

```python
for i in range(100):
    print(i, end=" ")  # 打印 0 - 99
```

### 4. 正确使用列表解析

```python
numbers = [i for i in range(10)]
```

### 5. Python中set的元素和dict的键值是可哈希的，因此查找起来时间复杂度为O(1)。

创建set引入的是一次性开销，创建过程将花费线性时间即使成员检查花费常数时间。因此如果你需要在循环里检查成员，最好先花时间创建set，因为你只需要创建一次。

```python
nlist = [1, 5, 4, 6]

nset = set(nlist)
for i in range(10):
    if i in nset:
        print(i)
```

### 6. 测试是否为空

```python
number = [1, 2, 3]
l_number = [i for i in number if i > 3]
if l_number:
    # Do something awesome
```

### 7. 测试是否为None 

```python
if x is not None:
    # Do something awesome
```

### 8. 测试变量是否为一些有用的值

```python
if x:
    # Do something awesome
```

### 9. 逻辑判断（比如if）时，Python当中等于False的值:

- 布尔型，False表示False，其他为True
- 整数和浮点数，0表示False，其他为True
- 字符串和类字符串类型（包括bytes和unicode），空字符串表示False，其他为True序列类型（包括tuple，list，dict，set等），空表示False，非空表示True
- None永远表示False


### 10. 到底什么是Python？你可以在回答中与其他技术进行对比（也鼓励这样做）。


下面是一些关键点：

- **Python是一种解释型语言**。这就是说，与C语言和C的衍生语言不同，Python代码在运行之前不需要编译。其他解释型语言还包括PHP和Ruby。
- **Python是动态类型语言**。指的是你在声明变量时，不需要说明变量的类型。你可以直接编写类似x=111和x="I'm a string"这样的代码，程序不会报错。
- **Python非常适合面向对象的编程（OOP）。** 因为它支持通过组合（composition）与继承（inheritance）的方式定义类（class）。Python中没有访问说明符（access specifier，类似C++中的public和private），这么设计的依据是“大家都是成年人了”。
- **函数是第一类对象（first-class objects）。** 这指的是它们可以被指定给变量，函数既能返回函数类型，也可以接受函数作为输入。**类（class）也是第一类对象**。
- **Python代码编写快，但是运行速度比编译语言通常要慢。** 好在Python允许加入基于C语言编写的扩展，因此我们能够优化代码，消除瓶颈，这点通常是可以实现的。numpy就是一个很好地例子，它的运行速度真的非常快，因为很多算术运算其实并不是通过Python实现的。
- Python用途非常广泛——网络应用，自动化，科学建模，大数据应用，等等。它也常被用作“胶水语言”，帮助其他语言和组件改善运行状况。
- Python让困难的事情变得容易，因此程序员可以专注于算法和数据结构的设计，而不用处理底层的细节

### dict的get():

```python
sum = {}
value = 'a'
sum[value] = sum.get(value, 0) + 1
```

### 其他的零碎

- 字符串切片，迭代器，with as  结构，in is关键字
- a = 1 if c else 2  等同其他语言的三元运算符。


---
