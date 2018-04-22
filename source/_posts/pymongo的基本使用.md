---
title: pymongo的基本使用
date: 2018-3-13
tags: [数据库, mongodb, python]
---

本文参考最新的 [pymongo 官方文档](http://api.mongodb.com/python/current/tutorial.html)进行整理，至文档更新时，**PyMongo的版本号为3.6.1**

## 先决条件

开始之前，请确保你的`pymongo`被正确安装，在 **Python shell** 中，应该运行以下代码而不会引发异常：

    import pymongo

另外一个条件就是确保MongoDB实例正在默认主机和端口上运行(如果你的操作系统是windows，可以参考[这篇文章](https://suadminwen.github.io/2018/01/25/mongodb%E7%9A%84windows%E6%9C%8D%E5%8A%A1%E5%AE%89%E8%A3%85/)进行mongodb的安装和运行)

<!--more-->

## 与MongoClient建立连接

使用 `PyMongo` 时的第一步是创建一个 `MongoClient`正在运行的 `mongod` 实例。这样做很简单：

```python
from pymongo import MongoClient
client = MongoClient()
```

上面的代码将连接到默认主机和端口。我们也可以明确指定主机和端口，如下所示：

    client = MongoClient("localhost", "27017")

或者使用MongoDB URI格式：

    client = MongoClient('mongodb://localhost:27017/')

## 获取数据库

一个 `MongoDB` 实例可以支持多个独立的 数据库。在使用 `PyMongo` 时，您可以使用 `MongoClient` 实例上的属性样式访问来访问数据库：

    db = client.test_database

如果您的数据库名称使用属性样式访问不起作用，则可以使用字典样式访问：

    db = client["test_database"]

## 获取集合

一个集合是一组存储在MongoDB中的文档，相当于关系型数据库中的表。在PyMongo中获取集合与获取数据库的工作方式相同：

    collection = db.test_collection
    collection = db["test_collection"]

**关于MongoDB中的集合（和数据库）的一个重要注意事项是它们是懒惰地创建的————上述命令都没有在MongoDB服务器上实际执行过任何操作。集合和数据库在第一个文档被插入时创建。**

## 文档的表示

MongoDB中的数据使用JSON样式的文档来表示（并存储）。在PyMongo中，我们使用字典来表示文档。例如，以下字典可能用于表示博客文章：

```python
import datetime
post = {"author": "Mike",
        "text": "My first blog post!",
        "tags": ["mongodb", "python", "pymongo"],
        "date": datetime.datetime.utcnow()}
```

请注意，文档可以包含Python类型（如datetime.datetime实例），这些类型 将自动转换为适当的 **BSON** 类型并从其中转换。

## 插入文档

要将文档插入到集合中，我们可以使用以下 `insert_one()` 方法：

```python
from pymongo import MongoClient
client = MongoClient()  # 连接数据库
db = client.blog  # 获取数据库
posts = db.posts  # 获取集合
post_id = posts.insert_one(post).inserted_id
print post_id  # ObjectId('...')
```

如果文档 "`_id`" 尚未包含密钥，则在插入文档时会自动添加一个特殊 "`_id`" 键。"`_id`"集合中的值必须是唯一的。`insert_one()` 返回一个实例 `InsertOneResult`。

插入第一个文档后，`posts` 集合实际上已经在服务器上创建。我们可以通过列出数据库中的所有集合来验证这一点：

    print db.collection_names(include_system_collections=False)  # [u'posts']

## 获取文档

可以在MongoDB中执行的最基本的查询类型是 find_one()。此方法返回与查询匹配的单个文档（或者None如果没有匹配项）。当您知道只有一个匹配的文档，或者只对第一个匹配感兴趣时，它非常有用。这里我们用来 find_one()从帖子集合中获取第一个文档：

    print posts.find_one()

结果是一个匹配我们先前插入的字典。

**注意 返回的文档包含一个"`_id`"插入时自动添加的文档。**

`find_one()`同时支持根据特定条件的查询。为了限制结果，我们现在只查询作者author为 **Mike** 的文档:

```python
print posts.find_one({"author": "Mike"})
# {u'date': datetime.datetime(2018, 3, 13, 6, 23, 51, 273000), u'text': u'My first blog post!', u'_id': ObjectId('5aa76e7d733f2d338cba01b5'), u'author': u'Mike', u'tags': [u'mongodb', u'python', u'pymongo']}
```

## 通过ObjectId查询

我们也可以通过它找到一个帖子_id，在我们的例子中是一个ObjectId:

```python
# 上面演示插入的时候，返回了一个post_id这个post_id的类型为 <class 'bson.objectid.ObjectId'>
print posts.find_one({"_id": post_id})

```

**请注意，ObjectId与其字符串表示形式不同：**

```python
post_id_as_str = str(post_id)
posts.find_one({"_id": post_id_as_str}) # No result
```

Web应用程序中的一项常见任务是从请求 URL 获取 ObjectId 并查找匹配的文档。在这种情况下，必须先将 ObjectId **从字符串转换为 ObjectId**，然后才能将其传递到 find_one()

```python
from bson.objectid import ObjectId
# The web framework gets post_id from the URL and passes it as a string
def get(post_id):
    # Convert from string to ObjectId:
    document = client.db.collection.find_one({'_id': ObjectId(post_id)})
```
## Unicode字符串

你可能会注意到返回结果的字符串与Python中默认的字符串有些不同(比如用u'Mike来代替'Mike')。这里简短说明一下。
MongoDB 以 BSON 格式存储数据，而 BSON 字符串使用的是 UTF-8 编码，所以 PyMongo 必须确保它存储的字符串为 UTF-8 格式。普通字符串( str )的存储不变，unicode 字符串会被 PyMongo 自动转为 UTF-8 格式。

## 批量插入

为了让查询更有趣，让我们插入更多的文档。除了插入单个文档之外，我们还可以通过将列表作为第一个参数传递给批量插入操作insert_many()。这会将每个文档插入列表中，只向服务器发送一个命令：

```python
new_posts = [{"author": "Mike",
              "text": "Another post!",
              "tags": ["bulk", "insert"],
              "date": datetime.datetime(2009, 11, 12, 11, 14)},
             {"author": "Eliot",
              "title": "MongoDB is fun",
              "text": "and pretty easy too!",
              "date": datetime.datetime(2009, 11, 10, 10, 45)}]
result = posts.insert_many(new_posts)
print result.inserted_ids  # [ObjectId('...'), ObjectId('...')]
```

有一下几点比较有趣的事情需要注意:
- 1. insert()的返回值包含了两个ObjectId对象，每个都对应上面批量插入的文档
- 2. new_posts[1] 与其他的posts看起来不一样:没有tags，并且增加了一个新的title。这里也证明了为什么我们一直说MongoDB是没有模式的

## 查询多个文档

作为查询的结果，我们使用该 find() 方法获取多个文档 。find()返回一个 Cursor实例，它允许我们迭代所有匹配的文档。例如，我们可以迭代posts集合中的每个文档：

```python
for post in posts.find():
    print post
```

find()也可以像find_one()那样来进行条件查询。现在，我们来查询所有作者author为Mike的文档:

```python
for post in posts.find({"author": "Mike"}):
    print post
```

## 计数

如果我们只想知道有多少文档匹配查询，我们可以执行count()操作而不是完整的查询。我们可以统计一个集合中的所有文档：

    posts.count()

或者只统计符合条件的:

    posts.find({"author": "Mike"}).count()

## 范围查询

MongoDB支持许多不同类型的高级查询。例如，让我们执行查询，将结果限制为比特定日期更早的帖子，还可以按作者对结果进行排序：

```python
d = datetime.datetime(2009, 11, 12, 12)
for post in posts.find({"date": {"$lt": d}}).sort("author"):
    print post
```

上面的代码中，我们使用了特殊操作符$lt来限制条件，并且使用了sort()方法来将结果以作者排序。


## 索引

添加索引可以帮助加速某些查询，还可以添加其他功能来查询和存储文档。在这个例子中，我们将演示如何在一个关键字上创建一个唯一的索引,该关键字拒绝该索引中已存在该关键字的文档。

首先，我们需要创建索引：

```python
result = db.profiles.create_index([('user_id', pymongo.ASCENDING)],unique=True)
print sorted(list(db.profiles.index_information()))
# [u'_id_', u'user_id_1']
```

请注意，我们现在有两个索引：一个是_id MongoDB自动创建的索引，另一个是user_id我们刚刚创建的索引。

现在让我们设置一些用户配置文件：

```python
user_profiles = [
    {'user_id': 211, 'name': 'Luke'},
    {'user_id': 212, 'name': 'Ziltoid'}]
result = db.profiles.insert_many(user_profiles)
```

索引阻止我们插入user_id已经在集合中的文档：

```python
new_profile = {'user_id': 213, 'name': 'Drew'}
duplicate_profile = {'user_id': 212, 'name': 'Tommy'}
result = db.profiles.insert_one(new_profile)  # This is fine.
result = db.profiles.insert_one(duplicate_profile)  # pymongo.errors.DuplicateKeyError: E11000 duplicate key error collection: blog.profiles index: user_id_1 dup key: { : 212 }
```

## 总结

其实pymongo的基本操作通mongodb类似，文档中只提到新增和查询。
