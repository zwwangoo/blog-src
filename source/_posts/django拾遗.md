---
title: django拾遗
date: 2017-12-1
tags: [python, django]
---

django是一个非常强大的web框架，虽然也是基于MVC构造的架构，但是更加关注模型(Model，数据存取层)、模板(Template，表现层)和视图(Views，业务逻辑层)，所以成为MTV模式。django的主要目的是简便、快捷的开发数据库驱动的网站：

- 对象关系映射(ORM): 以python类形式定义你的数据模型与关系数据库连接起来，你将得到一个非常容易使用的数据库API，同时也可以在django中使用原始的SQL语句。
- URL分派： 使用正则表达式匹配URL，你可以设计任意的URL，没有框架的特定限定。
- 模板系统： 使用django强大而可扩展的模板语言，可以分隔设计、内容和python代码。并且有可继承性。
- 表单处理： 你可以方便的生成各种表单模型，实现表单的有效检验。可以方便的从你定义的模型实例生成相应的表单。
- cache系统： 可以挂在内存缓存或者其他的框架实现超级缓存——实现你所需要的颗粒。
- 会话(session)： 用户登录与权限检查，快速开发用户会话功能。
- 国际化：内置国际化系统，方便开发出多种语言的网站。
- 自动化的管理界面：不需要你花大量的工作来创建人员管理和更新内容。Django自带一个ADMIN site,类似于内容管理系统

<!--more-->

以上是django的简单介绍。在本人的实际开发中，由于当时基础的薄弱和项目开发进度的追赶，虽然用了一年多的django，但是完全没有深入。项目已经上线，回头来认真学习django，才发现django原来是那么的强大。很多功能它已经帮助开发者完成，我们直接使用就可以，但是当时由于理解的浅显，之学到了皮毛。再次学习django，在这里总结一些django优秀或十分有用的地方。

# django模型（数据库）

这一部分，重点介绍一些django模型相关的知识，可以帮助我们更好的使用django模型和加快开发速度。

## Meta django的内部类。用于定义一些Django模型类的行为特性

- abstract

这个属性是定义当前的模型是不是一个抽象类。抽象类时不会对应数据库表的。一般我们用它来归纳一些公共属性字段，然后继承它的子类可以继承这些字段。取值为True或False。当abstract=True时，这个model就是一个抽象类。

- app_label

这个属性只在一种情况下使用，就是你的模型不在默认应用程序包下的models.py文件中，这时需要指定这个模型是哪个应用程序的。

- db_table

指定自定义数据库表名的。django有一套默认的按照一定规则生成数据模型对应的数据库表名，一般是应用的名称.models名。在MySQL中使用小写字母为表命名，当你通过db_table覆写表名称时，强烈推荐使用小写字母给表命名，特别是如果你用了MySQL作为后端。

- db_tablespace

当前模型所使用的数据库表空间 的名字。默认值是项目设置中的DEFAULT_TABLESPACE，如果它存在的话。如果后端并不支持表空间，这个选项可以忽略。

- ordering

这个字段是告诉Django模型对象返回的记录结果集是按照哪个字段排序的

```
ordering=['order_date'] # 按订单升序排列
ordering=['-order_date'] # 按订单降序排列，-表示降序
ordering=['?order_date'] # 随机排序，？表示随机
```

- verbose_name

就是给你的模型类起一个更可读的名字

- verbose_name_plural

这个选项是指定，模型的复数形式是什么

### 现学现用



```python
# -*- coding: utf-8 -*-
from __future__ import unicode_literals
from django.utils.encoding import python_2_unicode_compatible
from django.db import models


yes_or_no = (
    (True, "是"),
    (False, "否")
)


@python_2_unicode_compatible
class User(models.Model):
    name = models.CharField("姓名", max_length=10)
    email = models.EmailField("邮箱", unique=True)
    isUse = models.BooleanField("是否启用", choices=yes_or_no, default=True)

    def __str__(self):
        return self.name

    class Meta:
        verbose_name = "用户"
        verbose_name_plural = "用户"
        db_table = "user"


class Plan(models.Model):
    user = models.ForeignKey(User)
    content = models.CharField("计划内容", max_length=20)
    isDo = models.CharField("是否完成", choices=yes_or_no, max_length=5, default=False)
    description = models.CharField("备注", max_length=150, blank=True, null=True)
    notDoReason = models.CharField("未完成原因", max_length=150, blank=True, null=True)

    class Meta:
        abstract = True  # 这个model就是一个抽象类


class DayPlan(Plan):
    startTime = models.TimeField("开始时间")
    endTime = models.TimeField("结束时间")
    planDate = models.DateField("日期")

    class Meta:
        db_table = "dayPlan"
        verbose_name = "每日计划"
        verbose_name_plural = "每日计划"
        ordering = ["planDate"]


class WeekPlan(Plan):
    startDate = models.DateField("开始日期")
    endDate = models.DateField("结束日期")
    priority = models.IntegerField("优先等级", default=0)

    class Meta:
        db_table = "weekPlan"
        verbose_name = "每周要务"
        verbose_name_plural = "每周要务"
        ordering = ["endDate"]

```


## 创建对象的方法

一共有四种方法：

```
# 方法一

Author.objects.create(name="Jom", sex="男")

# 方法二

author = Author(name="Jom", sex="男")
author.save()

# 方法三

author = Author()
author.name = "Jom"
author.sex = "男"
author.save()

# 方法四，首先尝试获取，不存在就创建，可以防止重复

Author.objects.get_or_create(name="Jom", sex="男")

```

前三种方法返回的都是对应的 object，最后一种方法返回的是一个元组，(object, True/False)，创建时返回 True, 已经存在时返回 False

# Form


### 现学现用

```python

# views.py
def login(request):
    if request.method == "POST":
        fm = LoginForm(request.POST)
        if fm.is_valid():
            userName = fm.cleaned_data['userName']
            passWord = fm.cleaned_data['passWord']
            print userName, passWord
        return redirect(index)
    fm = LoginForm()
    return render(request, "login.html", {"fm": fm})

# forms.py
# -*- coding: utf-8 -*-
from django import forms


class LoginForm(forms.Form):
    userName = forms.CharField(max_length=20,
                               required=True,
                               widget=forms.TextInput(attrs={'class': 'weui-input',
                                                             'name': 'userName'}),
                               error_messages={'required': "用户名不能为空"})
    passWord = forms.CharField(max_length=20,
                              required=True,
                              widget=forms.PasswordInput(attrs={'class': 'weui-input',
                                                                'name': 'passWord'}),
                              error_messages={'required': "密码不能为空"})


```
