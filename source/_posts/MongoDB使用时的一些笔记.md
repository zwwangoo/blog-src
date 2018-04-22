---
title: MongoDB使用时的一些笔记
date: 2018-3-13 17:09
tags: [数据库, mongodb]
---

首先保证mongodb正确安装，并启动服务[这篇文章](https://suadminwen.github.io/2018/01/25/mongodb%E7%9A%84windows%E6%9C%8D%E5%8A%A1%E5%AE%89%E8%A3%85/)讲解了windows下进行mongodb的安装和运行，启动mongo的命令行，打开一个命令行窗口，执行如下命令即可

    mongo

## 转到用到的数据库

先用 mongo 命令行连接到一个 MongoDB 实例，转到 test 数据库。

    use test

<!--more-->

## 范围查询

- 大于操作(`$gt`)
- 大于或等于(`$gte`)
- 小于操作(`$lt`)
- 小于或等于(`$lte`)
- 逻辑AND(`,`或`$and`)
- 逻辑OR(`$or`)

```
db.users.find({"grade": {"$gt": "12"}})
db.users.find({"$or": [{"name": "a"}, {"grade": "12"}]})
db.users.find({"$or": [{"name": "a"}, {"grade": {"$lt":"12"}}]})
db.users.find({"name": "a", "grade": "12"})
db.users.find({"$and": [{"name": "a"},{"grade": "12"}]})
```

## 排序查询结果

指定查询结果排序方式的就是在查询后追加一个 `sort()` 方法调用。传递给此方法一个文档，包含指定排序字段和排序类型。1表示升序，-1表示降序。

```
db.users.find({"grade": {"$gt": "12"}}).sort({"grade": 1, "name": -1})
```

## 更新

`update(<filter>, <update>, <options>)` 方法更新文档，`update()` 方法只更新一个文档。如果要更新多个文档，需要指定 `multi` 可选参数。

`$set` 用来修改值。如果字段不存在，`$set` 会创建这个字段。

```
db.users.update({"name": "a"}, {"$set": {"grade":"1000"}}, {multi: true})
```

## 替换

要替换一个文档，只需要把一个新的文档传递给`update()`的第二个参数，并且不需要包含`_id`字段。如果包含`_id`字段，只保证跟原文档是同一个值。用于替换的文档可以跟原文档具有完全不同的字段。

```
db.users.update({"name": "a"}, {"name": "aa", "grade": "1200"})
```

## 删除

使用`remove()`方法从集合中删除文档。这个方法需要一个条件文档用来决定哪些文档将被删除。

默认地，`remove()`方法将删除匹配指定条件的所有文档。使用`justOne`可选参数可以限制删除操作只删除一条。

```
db.users.remove({"name": "a"}, {justOne: true})
```

删除所有文档：删除所有的操作仅仅是删除集合中的全部文档。集合本身和集合的索引并不会被删除。

```
db.users.remove({})
```

删除一个集合:

```
db.users.drop()
```

在MongoDB中，"写"操作是文档级别的原子操作。如果一个删除操作要删除集合中的多个文档，这个操作会和其他写操作交错。

## Limit() 方法 和 Skip() 方法

- `limit()`方法来读取指定数量的数据
- `skip()`方法来跳过指定数量的数据，`skip()`方法同样接受一个数字参数作为跳过的记录条数。

```
db.users.find().limit(2)
db.users.find().skip(2)
```
**当查询时同时使用`sort`,`skip`,`limit`，无论位置先后，最先执行顺序 `sort`再`skip`再`limit`。**
**skip和limit方法只适合小数据量分页，如果是百万级效率就会非常低，因为`skip`方法是一条条数据数过去的**
