---
title: Python开发人员犯下的10个最常见的错误
date: 2018-12-21
tags: [python]
---

# 滥用表达式作为函数参数的默认值

python允许通过为函数提供默认值来指定函数参数的，但是当默认值是可变的时，就会产生一些问题：

```python
def foo(bar=[]):
	bar.append('baz')
	return bar
```

上面的代码中，期望的是 `foo()` 重复调用（即不指定bar参数）将始终返回 `'baz'` ，因此假设每次 `foo()` 调用 `bar` 被设置为 `[]`。

但是，让我们来看看执行次操作时实际发生的情况：

```shell
>>> foo()
['baz']
>>> foo()
['baz', 'baz']
```

咦，为什么每次调用都会默认值附加 `'baz'` 到现有的列表中，而不是每次都创建一个新列表？

答案就是： **函数参数的默认值仅在定义函数时计算一次**。因此 `bar` 仅在 `foo()` 首次定义时将参数初始化为其默认值，但随后调用 `foo()` (即未指定`bar`参数)，将继续使用 `bar` 最初初始化的相同列表。

仅供参考，一个常见的解决方法如下：

```
def foo(bar=None):
	if not bar:
	    bar = []
	bar.append('baz')
	return bar
```

<!--more-->

# 错误的使用类变量

请考虑一下示例：

```python
>>> class A(object):
...     x = 1
... 
>>> class B(A):
...     pass
... 
>>> class C(A):
...     pass
... 
>>> print(A.x, B.x, C.x)
1 1 1
```

以上的输出是没有问题的，请继续往下看：

```python
>>> B.x = 2
>>> print(A.x, B.x, C.x)
1 2 1
```

输出还是如预期的那样，那接下来：

```
>>> A.x = 3
>>> print(A.x, B.x, C.x)
```

可以思考一下上面输出的结果：

`3 2 3`

这是什么情况？我们只改变了A.x，为什么C.x也改变了呢？


在python中，类变量在内部作为字典处理，并遵循通常成为方法解析顺序(MRO)的方法，因此在上面的代码中，由于在`C`中找不到`x`属性，因此将在其基类中查找它。换句话数，`C`没有自己的`x`属性，因此引用C.x实际上值得是A.x。


# 错误的为异常块指定参数

假如你用一下代码：

```python
# 这段代码是python 2.7版本的
>>> try:
...     l = ["a", "b"]
...     int(l[2])
... except (ValueError, IndexError):  # To catch both exceptions, right?
...     pass
...
Traceback (most recent call last):
  File "<stdin>", line 3, in <module>
IndexError: list index out of range
```

这里的问题是except语句没有采用这种方式指定的异常列表，相反，在python2.x中，语法 `except Exception, e` 用于将异常绑定到指定的可选的第二个参数(本例中e)，以用于进一步检查。结果在上面的代码中，IndexError异常没有被`except`语句捕获，相反，异常最终被绑定到一个名为`IndexError`的参数。

在`except`语句中，捕获多个异常的正确方法是将第一个参数指定为包含要捕获所有异常的元祖，此外为了获得最大的可移植性，请使用`as`关键字，因为Python2和Python3都支持该语法。

```python
>>> try:
...     l = ['a', 'b']
...     int(l[2])
... except (ValueError, IndexError) as e:
...     pass
... 
>>> 
```

# 误解Python范围规范

Python范围解析是基于所谓的LEGB规则。在Python的工作方式中有一些细微之处，让我们看看常见的更高级的Python编程问题：

```
>>> x = 10
>>> def foo():
...     x += 1
...     print(x)
...
>>> foo()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 2, in foo
UnboundLocalError: local variable 'x' referenced before assignment
```

上述问题出现的原因是：当你对作用域中的变量进行赋值时，Python会自动将变量视为该作用域的本地变量，并在任何外部作用域中隐藏任何类似命名的变量。

但在使用列表时，有一个特殊的现象，请看以下代码示例：

```python
>>> lst = [1, 2, 3]
>>> def foo1():
...     lst.append(5)
...
>>> foo1()
>>> lst
[1, 2, 3, 5]
>>> lst = [1, 2, 3]
>>> def foo2():
...     lst += [5]
...
>>> foo2()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 2, in foo2
UnboundLocalError: local variable 'lst' referenced before assignment
```

咦？为什么foo1良好的运行，但是foo2却报错了？？

答案和前面示例问题相同，但无疑更微妙一些。`foo1`不是分配值到lst，而foo2却是。记住`lst += [5]`实际是`lst = lst + [5]`的简写，我们看到foo2正在分配一个值给lst，因此Python推测它是本地范围内。但是我们要分配的值lst是lst自身，因此是未定义。


# 在迭代时修改列表

以下代码的问题是相当明显的：

```
>>> odd = lambda x: bool(x % 2)
>>> numbers = [n for n in range(10)]
>>> for i in range(len(numbers)):
...     if odd(numbers[i]):
...         del numbers[i]
...
Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
IndexError: list index out of range
```

在迭代时，从列表或数组中删除项是Python常见的问题。幸运的是Python结合许多优雅的编程范例，如果使用得当可以简化代码。另外一个好处是更简单的代码不太可能被意外删除列表项而导致迭代问题。它完美的工作：

```
>>> odd = lambda x : bool(x % 2)
>>> numbers = [n for n in range(10)]
>>> numbers[:] = [n for n in numbers if not odd(n)]  # ahh, the beauty of it all
>>> numbers
[0, 2, 4, 6, 8]
```

# 混淆Python如何绑定闭包中的变量

参考以下示例：

```python
>>> def create_multipliers():
...     return [lambda x: i * x for i in range(5)]
...
>>> for multiplier in create_multipliers():
...     print(multiplier(2))
...
```

你可能期望以下输出：

0
2
4
6
8

但是你得到的是：

8
8
8
8
8

!!

这是因为Python调用内部函数时，闭包中使用的变量值是**后期绑定**行为导致的。所以上面的代码中，每当调用任何返回的函数时，在调用i它时，在周围的作用域中查找值，那是循环已经完成，因此i已经分配了它的最终值4。

这个常见问题的解决是有点像黑客的做法：

```python
>>> def create_multipliers():
...     return [lambda x, i=i : i * x for i in range(5)]
...
>>> for multiplier in create_multipliers():
...     print(multiplier(2))
...
0
2
4
6
8
>>>
```

这里利用了默认参数来生成匿名函数，以实现所需的行为，有些人称之为优雅，有些人会认为微免，有些人会讨厌它。但是作为Python的开发人员，无论如何都要理解它。


# 创建循环模块依赖项

假设你有两个文件，`a.py` 和`b.py` 而且每个文件都导入另一个文件，如下所示：

在`a.py`中：

```python
import b

def f():
	return b.x

print(f())
```

在`b.py`中：

```python
import a

x = 1

def g():
	print(a.f())
```

首先让我们尝试导入`a.py`

```
>>> import a
1
```

到此，没有出现异常，也许这个会给你带来惊喜，毕竟，我们这里有一个循环导入的问题，大概应该是一个问题，不应该？答案是，仅仅存在循环导入本身并不是Python的一个问题。如果已导入的模块，Python足够聪明，不会尝试重新导入它。但是根据每个模块尝试访问另一个模块中定义的函数或变量，你可能会遇到一些问题。

所以回到例子中，当我们导入`a.py`，它导入`b.py`有没有问题？因为`b.py` 不需要从`a.py`中导入任何变量，这是因为唯一调用的`a.f()`还是在调用`g()`时被调用，所以此时`a.py`或`b.py`中没有任何内容调用`g()`，所以一切看起来是美好的。


如果我们尝试导入`b.py`，看看会发生什么？(前提是没有先导入`a.py`)

```
>>> import b
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/home/wen/b.py", line 1, in <module>
    import a
  File "/home/wen/a.py", line 8, in <module>
    print(f())
  File "/home/wen/a.py", line 5, in f
    return b.x
AttributeError: module 'b' has no attribute 'x'
>>>
```

这就出现问题了。 在导入`b.py`中，他会尝试导入`a.py`,而后者又会调用`f()`尝试访问的内容`b.x`，但`b.x`尚未定义，因此出现`AttributeError`问题。

这里提供一个简单的方案处理这个问题，只需要修改`b.py`，在`g()`中导入`a.py`：

```python
x = 1
def g():
	import a
	print(a.f())
```

当我们导入它时，一切都会变得美好：

```python
>>> import b
>>> b.g()
1
1
```

# 8 名称与Python标准库模块冲突

Python的优点之一是它提供了“开箱即用”的丰富的库模块。但是如果你没有意识的避开它，那么在发成自定义模块与Python标准库模块冲突的几率会增大很多。

# 9 未能解决Python2和Python3之间的差异

考虑一下文件 `foo.py`

```python
import sys

def bar(i):
    if i == 1:
        raise KeyError(1)
    if i == 2:
        raise ValueError(2)

def bad():
    e = None
    try:
        bar(int(sys.argv[1]))
    except KeyError as e:
        print('key error')
    except ValueError as e:
        print('value error')
    print(e)

bad()
```

在Python2上，正常运行：

```python
$ python foo.py 1
key error
1
$ python foo.py 2
value error
2
```

但是在Python3上：

```python
$ python3 foo.py 1
key error
Traceback (most recent call last):
  File "foo.py", line 19, in <module>
    bad()
  File "foo.py", line 17, in bad
    print(e)
UnboundLocalError: local variable 'e' referenced before assignment
```


“问题”是，在Python 3中，异常对象超出except块的范围是不可访问的。（原因是，否则，它会在内存中保持堆栈帧的引用循环，直到垃圾收集器运行并从内存中清除引用。

避免此问题的一种方法是在块的范围之外维护对异常对象的引用，以except使其保持可访问状态。这是使用此技术的上一个示例的一个版本，从而产生兼容Python 2和Python 3的代码：

```python
import sys

def bar(i):
    if i == 1:
        raise KeyError(1)
    if i == 2:
        raise ValueError(2)

def good():
    exception = None
    try:
        bar(int(sys.argv[1]))
    except KeyError as e:
        exception = e
        print('key error')
    except ValueError as e:
        exception = e
        print('value error')
    print(exception)

good()
```

在Python3上运行：

```python
$ python3 foo.py 1
key error
1
$ python3 foo.py 2
value error
2
```

# 10 滥用`__del__`方法

假设你在一个名为的文件中有这个`mod.py`：

```
import foo

class Bar(object):

	...

	def __del__(self):
		foo.cleanup(slef.myhandle())
```

然后你试着这样做 `another_mod.py`:

```
import mod
mybar = mod.Bar()
```

你会得到一个丑陋的`AttributeError`。

当解释器关闭时，模块的全局变量都被设置为None。因此，在上面的示例中，在`__del__`调用的位置，名称`foo`已设置为`None`。

解决方案是使用`atexit.register()`。这样，当您的程序完成执行时（正常退出时），您的注册处理程序将在解释器关闭之前启动:

```
import foo
import atexit

def cleanup(handle):
    foo.cleanup(handle)


class Bar(object):
    def __init__(self):
        ...
        atexit.register(cleanup, self.myhandle)
```

此实现提供了一种干净可靠的方法，可在正常程序终止时调用任何所需的清理功能。显然，foo.cleanup要决定如何处理绑定到名称的对象self.myhandle，但是你明白了。


[原文](https://www.toptal.com/python/top-10-mistakes-that-python-programmers-make)
