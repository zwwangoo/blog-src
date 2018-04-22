---
title: python网络爬虫和信息提取(3)
date: 2017-12-17
tags: [python, 网络爬虫]
---

## re库的使用和爬虫实战

- 通用的字符串表达框架
- 简洁表达一组字符串的表达式
- 针对字符串表达“简洁”和“特征”思想的工具
- 判断某字符串的特征归属

正则表达式在 **文本处理中** 的应用：

- 表达文本类型的特征（病毒、入侵等）
- 同时查找或替换字符字符串
- 匹配字符串的全部或部分

**主要用在字符串匹配中**

<!--more-->

正则表达式的使用：

编译：将符合正则表达式语法的字符串转换成正则表达式特征。编译之前的表达式只是一个符合正则表达式语法的字符串。


### 正则表达式的语法

- 正则表达式的常用操作符

| 操作符 | 说明 | 实例 |
| :------------- | :------------- | :------------- |
| `.` | 表示任何单个字符 |  |
| `[]` | 字符集，对单个字符给出取值范围 | `[abc]`表示a、b、c,`[a-z]`表示a到z单个字符 |
| `[^ ]` | 非字符集，对单个字符给出排除范围 | `[^abc]`表示非a或b或c的单个字符 |
| `*` | 前一个字符0次或无限次扩展 | `abc*`表示ab、abc、abcc、abccc等 |
| `+` | 前一个字符1次或无限次扩展 | `abc+`表示abc、abcc、abccc等 |
| `?` | 前一个字符0次或1次扩展 | `abc?`表示ab、abc |
| `｜` | 左右表达式任意一个 | `abc｜def` 表示abc、def |
| `{m}` | 扩展前一个字符m次 | `ab{2}c`表示abbc |
| `{m, n}` | 扩展前一个字符m至n次（含n） | `ab{1, 2}c`表示abc、abbc |
| `^` | 匹配字符串开头 | `^abc`表示abc且在一个字符串的开头 |
| `$` | 匹配字符串结尾 | `abc$`表示abc且在一个字符串的结尾 |
| `()` | 分组标记，内部只能只用｜操作符 | `(abc)`表示abc,`(abc｜def)`表示acb、def |
| `\d` | 数字，等价与`[0-9]` ||
| `\w` | 单词字符，等价于`[A-Za-z0-9]` ||

- 语法实例：

| 正则表达式 | 对应字符串 |
| :------------- | :------------- |
| `P(Y｜YT｜YTH｜YTHO)?N` | 'PN'、'PYN'、'PYTN'、'PYTHN'、'PYTHON' |
| `PYTHON+` | 'PYTHON'、'PYTHONN'、'PYTHONNN'、…… |
| `PY[TH]ON` | 'PYTON'、'PYTHON' |
| `PY[^TH]?ON` | 'PYON'、'PYN'、'PYbN'、…… |
| `PY{:3}N` | 'PN'、'PYN'、'PYYN'、'PYYYN' |

- 经典正则表达式实例

| 表达式 | 说明 |
| :------------- | :------------- |
| `^[A-Za-z]+$` | 由26个字母组成的字符串 |
| `^-?\d+$` | 整数形式的字符串 |
| `[\u4e00-\u9fa5]` | 匹配中文字符 |
| `^\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$` | Email地址 |
| `(^\d{15}$)｜(^\d{18}$)｜(^\d{17}(\d｜X｜x)$)` | 身份证号 |

- 正则表达式运算优先级

![正则表达式运算优先级](https://i.loli.net/2017/12/18/5a3730644bf23.jpg)

### Re使用

re库是python的标准库，主要用于字符串匹配。调用方法很简单： `import re`。re库采用raw string(原生字符串，不包含转义符的字符串)类型表示正则表达式，表示为`r'text'`

**re库主要功能函数**

![re库主要功能函数](https://i.loli.net/2017/12/18/5a37306458664.jpg)

flags：正则表达式使用时的控制标记

| 常用标记 | 说明 |
| :------------- | :------------- |
| re.I re.IGNORECASE | 忽略正则表达式的大小写，｜A-Z｜能够匹配小写字符 |
| re.M re.MULTILINE | 正则表达式中的^操作符能够将给定字符串的每行当作匹配开始 |
| re.S re.DOTALL | 正则表达式中的.操作符能够匹配所有字符，默认匹配除换行外的所有字符 |

**re库的最小匹配和贪婪匹配**

```python
math = re.search(r'PY.*N', 'PYANBNCNDN')
math.group(0)  # PYANBNCNDN
```

re库默认使用贪婪匹配模式。如果想返回最小匹配结果：

```python
math = re.search(r'PY.*?N', 'PYANBNCNDN')
math.group(0)  # PYAN
```

| 最小匹配 | 说明 |
| :------------- | :------------- |
| *? | 前一个字符0次或无限次扩展，最小匹配 |
| +? | 前一个字符1次或无限次扩展，最小匹配 |
| ?? | 前一个字符0次或1次扩展，最小匹配 |
| {m, n} | 扩展前一个字符m至n次(含n)，最小匹配 |

---

### 淘宝商品比价定向爬虫

**功能描述**

- 目的：获取淘宝搜索页面的信息，提取其中的商品名称和价格
- 理解：淘宝的搜索接口、翻页的处理
- 技术路线：requests+re

**url分析**

首页提交搜索：

`url = 'https://s.taobao.com/search?q=书包&imgfile=&commend=all&ssid=s5-e&search_type=item&sourceId=tb.index&spm=a21bo.2017.201856-taobao-item.1&ie=utf8&initiative_id=tbindexz_20170306'`

搜索结果的第二页：

`url = 'https://s.taobao.com/search?q=书包&imgfile=&commend=all&ssid=s5-e&search_type=item&sourceId=tb.index&spm=a21bo.2017.201856-taobao-item.1&ie=utf8&initiative_id=tbindexz_20170306&bcoffset=4&ntoffset=4&p4ppushleft=1%2C48&s=44'`

**程序设计**

- 步骤1：提交商品搜索请求，循环获取页面
- 步骤2：对于每个页面，提取商品名称和价格信息
- 步骤3：将信息输出到屏幕上

---

```python
# -*- coding: utf-8 -*-
import requests
import re

def getHtmlText(url):
    try:
        r = requests.get(url, timeout=30)
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        return r.text
    except:
        return ''


def parsePage(ilt, html):
    try:
        plt = re.findall(r'\"view_price\"\:\"[\d\.]*\"', html)
        tlt = re.findall(r'\"raw_title\"\:\".*?\"', html)
        for i in range(len(plt)):
            price = eval(plt[i].split(':')[1])
            title = eval(tlt[i].split(':')[1])
            ilt.append([price, title])
    except:
        print 'error'


def printGoodsList(ilt):
    tplt = "{:4}\t{:8}\t{:16}"
    print tplt.format('序号', '价格', '商品名称')
    count = 0
    for g in ilt:
        count = count +1
        print tplt.format(count, g[0], g[1])
    print ''


def main():
    goods = '书包'
    depth = 2
    start_url = 'https://s.taobao.com/search?q=' + goods
    inforList = []
    for i in range(depth):
        try:
            url = start_url + '&s=' + str(44*i)
            html = getHtmlText(url)
            parsePage(inforList, html)
        except:
            continue
    printGoodsList(inforList)


if __name__ == '__main__':
    main()
```
---
![运行结果](https://i.loli.net/2017/12/18/5a377e467083a.jpg)
