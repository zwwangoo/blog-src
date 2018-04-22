---
title: python数据结构及常用排序算法
date: 2017-12-03
tags: [python, 数据结构, 排序算法]
---

这里总结些python中的数据结构和常用的排序算法。

## 数据结构

### 栈

在python中并没有栈这个数据类型，所以需要自己进行定义和封装。其实也就是通过列表来实现栈的操作。这里就定义了一个栈和栈的基本操作。
但是需要注意的是：这里使用list作为栈，是没有进行同步的，特别是在多线程中，可能会出现并发的问题，这里有待验证。

<!--more-->

```python
class Stack(objects):
    def __init__(self):
        self.value = []

    def push(self, value):
        self.value.append(value)

    def pop(self):
        if self.isEmpty():
            raise Exception("Your stack is empty")
        return self.value.pop()

    def peek(self):
        # 返回栈顶元素
        if self.isEmpty():
            raise Exception("Your stack is empty")
        return self.value[0]

    def isEmpty(self):
        return self.size() == 0

    def size(self):
        return len(self.value)

```

### 树

二叉树的创建：

```python
# -*- coding: utf-8 -*-


class Node(object):
    def __init__(self, value):
        self.value = value
        self.lChild = None
        self.rChild = None


def createBinaryTree():
    # 先序递归的方式创建二叉树，
    # 按先后次序输入二叉树中的节点，以#表示空树

    value = raw_input("请输入当前节点的值（如果是空树，请用“#”表示）> ")
    if value == "#":
        root = None  # 当前节点为空
    else:
        root = Node(value)
        root.lChild = createBinaryTree()
        root.rChild = createBinaryTree()

    return root


if __name__ == "__main__":
    root = createBinaryTree()
    print "先序创建二叉树完毕！"

```


### 链表

链表的创建和打印：

```python
# -*- coding:utf-8 -*-
class ListNode(objects):
   def __init__(self, x):
       self.value = x
       self.next = None


def createList():
    # 创建链表 9->8->7->6->5->4->3->2->1->0
    head = None
    for i in range(10):
        node = ListNode(i)
        node.next = head
        head = node
    return head


def printfList(head):
    # 打印链表
    while head:
        print head.value
        head = head.next
```

#### 算法题1

输入两个 **单调递增** 的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足 **单调不减** 规则。[牛客网《剑指offer》](https://www.nowcoder.com/practice/d8b6b4358f774294a89de2a6ac4d9337?tpId=13&tqId=11169&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)


```python
# 解体思路来自左程云《程序员代码面试指南》P84
def merge(pHead1, pHead2):

    # 如果两个链表有一个是空的，那么就直接返回另外一个的头部
    if pHead1 == None: return pHead2
    if pHead2 == None: return pHead1

    # 将链表中最小值的节点赋值给head
    if pHead1.value > pHead2.value:
        head = pHead2
    else:
        head = pHead1

    # 领最小值节点所在的那个链表为cur1,另外一个为cur2
    if head == pHead1:
        cur1 = pHead1
        cur2 = pHead2
    else:
        cur1 = pHead2
        cur2 = pHead1

    preNode = None  # 这个节点表示上一次比较中较小的值
    nextNode = None  # 这个节点表示下一个要处理的节点

    while cur1 and cur2:
        if cur1.value <= cur2.value:
            preNode = cur1
            cur1 = cur1.next
        else:
            nextNode = cur2.next
            preNode.next = cur2
            cur2.next = cur1
            preNode = cur2
            cur2 = nextNode
    preNode.next = cur1 or cur2
    return head
```


## 算法

### 常见排序算法

- 冒泡排序(O^2): 冒泡排序是利用两层循环，不断地将数据中比较大或比较小的数字交换到最上面来，看上去就像水中的气泡冒出来一样。

冒泡排序是稳定的。以下提供三种方法，第一种是将小数先冒出来，第二种是将大数先冒出来，第三种提供的是一种优化思路：设置了一个标志，某一趟排序时并没有进行数据交换，则说明所有数据已经有序。这样做是优化了当数组有序的时候，进行的不必要遍历


```python
def bubble_soft_1(data):
    """冒泡排序（升序）
    这个排序的过程是先排小数
    :param data: 待排序的数组
    :return: 排完序的数组
    """
    length = len(data)
    for i in range(length - 1):  # 这里结束的位置是减一的，因为最后一趟已经是有序的了，不需要再遍历
        for j in range(i + 1, length):  # 这里是从下一个位置开始
            if data[i] > data[j]:
                data[j], data[i] = data[i], data[j]
    return data


def bubble_soft_2(data):
    """冒泡排序（升序）
    这个排序的过程是先排大数
    :param data: 待排序的数组
    :return: 排完序的数组
    """
    length = len(data)
    for i in range(length - 1):  # 注意这里的循环变量控制
        for j in range(length - 1):  
            if data[j + 1] < data[j]:
                data[j], data[j + 1] = data[j + 1], data[j]
    return data


def bubble_soft_3(data):
    """冒泡排序（升序）优化
    这个排序的过程是先排大数，
    设置了一个标志，某一趟排序时并没有进行数据交换，则说明所有数据已经有序
    这样做是优化了当数组有序的时候，进行的不必要遍历
    :param data: 待排序的数组
    :return: 排完序的数组
    """
    length = len(data)
    for i in range(length - 1):
        change = False  # 这是一个标识位
        for j in range(length - 1):
            if data[j + 1] < data[j]:
                data[j], data[j + 1] = data[j + 1], data[j]
                change = True
        if not change:
            break
    return data

```

- 选择排序(O^2):每一次循环从乱序部分中寻找出最大的那个数字，交换到有序部分的最后去，直到列表都有序。

```python
def selection_sort(data):
    # 选择排序
    length = len(data)

    for i in range(length):
        index = i
        for j in range(i+1, length):
            if data[index] < data[j]:
                index = j  # 记录当前循环中最大的记录
        if i == index: continue
        data[index], data[i] = data[i], data[index]
    return data
```

总共的运行次数和冒泡排序一样，n(n-1)/2次，但是由于交换次数少得多了，速度也就更快了。


- 插入排序(O^2)： 将每一个无序部分的元素插入到有序部分中。

```python
def insertion_sort1(data):
    # 插入排序实现1
    length = len(data)
    for i in range(1, length):  # 默认第一个元素是有序的
        now_num = data[i]
        j = 0  # 从前面查找插入
        while data[j] > now_num and j < i:
            j += 1
        if j == i: continue
        data = data[:j] + [now_num] + data[j:i] + data[i + 1:]
    return data


def insertion_sort2(data):
    # 插入排序实现2
    length = len(data)

    for i in range(1, length):
        now_num = data[i]
        j = i -1  # 从后面查找插入
        while j >= 0 and data[j] < now_num:
            data[j+1] = data[j]
            j -= 1
        data[j + 1] = now_num
    return data

# 插入排序再考虑一种利用链表是的排序效率

```

在数组元素比较少的时候，用几行就能实现的这个算法看上去很简洁，但一旦数量上去了，每一轮插入需要比较的次数越来越多，拖慢了速度。

插入排序的两种实现方式的时间复杂度相同，但是后面一种方法的代码复杂度比较低

- 希尔排序(n^2)

```python
def shell_sort(data):
    """
    希尔(Shell)排序: 缩小增量排序，它是一种插入排序。
    它是直接插入排序算法的一种威力加强版
    :param data:
    :return:
    """
    length = len(data)
    step = length // 2  # 初始步长
    while step > 0:
        # 其实下面执行的就是插入排序啦，只不过需要注意步长
        for i in range(step, length):  # 每一列进行插入排序 , 从step 到 n-1
            now_num = data[i]
            j = i
            while j >= step and data[j - step] > now_num:  # 插入排序
                data[j] = data[j - step]
                j -= step
            data[j] = now_num
        step //= 2  # 重新设置步长
    return data
```


- 快速排序(nlogn)


快速排序是对冒泡排序的一种改进。通过一趟排序将要排序的数据分割独立的两部分，一部分所有数据都比另一部分的所有数据都要小，然后再按照此方法对这两个部分分别进行快速排序，整个过程可以递归进行，以达到整个数据变成有序序列。


首先选取一个数据(通常选用数组的第一个数作为关键数据)，然后所有比它小的数都放到它前面，所有比它大的数都放到它后面，这个过程成为一趟快速排序。


```python
def quick_sort(data):
  	if len(data) <= 1: return data
  	pivot = data[0]

  	left = quick_sort([x for x in data[1:] if x <= pivot])
  	right = quick_sort([x for x in data[1:] if x > pivot])

  	return left + [pivot] + right
```

快速排序也不是稳定的。最坏的情况下复杂度也达到: O(n^2)
