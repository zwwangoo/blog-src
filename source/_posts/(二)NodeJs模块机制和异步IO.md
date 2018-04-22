---
title: (二)NodeJs模块机制和异步IO
date: 2017-10-23
tags: [NodeJs, 阅读笔记]
---

> 说明：该学习笔记参考《深入浅出Node.js》在学习过程中，添加了自己的理解和适当的补充！仅供参考！

# NodeJsmo模块机制

## CommonJs的模块规范

### 模块引用

示例代码如下：

```js
var math = require("math");
```

在CommonJs规范中，存在`require()`方法，这个方法接受模块标识，以此引入一个模块的API到当前上下文中。

<!--more-->

### 模块定义

在模块中，上下文提供`require()`方法来引入外部模块。对应引入的功能，上下文提供了`exports`对象用于导出当前模块的方法或者变量，并且它是唯一到处的出口。在模块中，还存在一个`module`对象，它代表模块自身，而`exports`是`module`的属性。在NodeJs中一个文件就是一个模块，将方法挂载在`exports`对象上可作为属性即可定义导出的方式：

```js
//math.js

exports.add = function(){
	var sum = 0,
	i = 0,
	args = arguments,
	l = args.length;
	while(i < 1){
		sum += args[i++];
	}
	return sum;
}
```
在另一个文件中通过require()方法引入模块后，就能调用定义的属性和方法了，

```js
//program.js
var math = require("./math");
exports.iscrement = function(val){
	return math.add(val, 1);
};
```

这里，`math.js`和`program.js`在同一级目录下，最后在`program.js`中`require("./math")`。

### 模块标识

模块标识其实就是通过传递给`require()`方法的参数，它必须是符合小驼峰命名的字符串，或者是以`.`、`..`开头的相对路径，或者是绝对路径，它可以没有文件后缀.js

每个模块具有独立的空间，它们相互不干扰，在引用时也显得干净利落。这套模块导出和引用机制使得用户完全不必考虑变量污染，命名空间等。

## NodeJs的模块实现

在NodeJs中引入模块，需要经历以下三个步骤：

- 1 路径分析
- 2 文件定位
- 3 编译执行

模块分为两大类：

- 核心模块，有NodeJs提供的模块。
- 文件模块，有用户编写的模块。

### 优先从缓存加载

NodeJs对引入过的模块都会进行缓存，缓存的是编译和执行之后的对象。

不管是核心模块还是文件模块，`require()`方法对相同模块的二次加载都一律采用缓存优先。不同的是核心模块的缓存先于文件模块的缓存检查。

### 路径分析和文件定位

**模块标识符**主要分为以下几类：

- 核心模块，如`http`，`fs`， `path`
- `.`或者`..`开始的相对路径文件模块
- 以`/` 开始的绝对路径文件模块
- 非路径形式的文件模块，如自定义的`connect`模块自定义模块

如果试图加载一个与核心模块标识符相同的，那是不会成功的。想要加载成功，必须选择一个不同的标识符或者换用路径的方式。

自定义模块的查找是最费时间的。

模块路径是NodeJs在定义文件模块的具体文件时定制的查找策略，具体表现为一个路径组成的数组。它的生成方式与JavaScript的原型链或作用域链的查找方式十分类似。在加载的的过程中，NodeJs会逐个尝试模块路径中的路径，直到找到目标文件为止。

```js
// moudle_path.js
console.log(module.paths);
```
**文件定位**
require()在分析标识符的过程中，会出现标识符中不包含文件扩展名的情况。CommonJs模块规范也允许标识符中不包含文件扩展名，这种情况下，NodeJs会按`.js` 、`.json`、`.node`的次序补足扩展名，依次尝试。


![文件定位流程图](http://upload-images.jianshu.io/upload_images/3248493-7e36a65fa76a1983.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


在尝试的过程中，需要调用fs模块同步阻塞式地判断文件是否存在，由于NodeJs是单线程的，所以这里会引起一个性能问题。

**如果是`.node`和`.json`文件，在传递给`require()`的标识符中带上扩展名，会加快一些速度。**
**同步配合缓存，可以大幅度缓解NodeJs单线程中阻塞式调用的缺陷。**


在分析标识符的过程中，`require()`通过分析文件扩展名之后，可能没有查找到对应文件，但却得到一个目录，这在引入自定义模块和逐个模块路径精心查找时经常会出现，此时NodeJs会将目录当成一个包来处理。

### 模块编译

在NodeJs中，每个文件都是一个对象，它的定义如下：

```js
function Module(id, parent){
    this.id = id;
    this.exports = {};
    this.parent = parent;
    if (parent && parent.children){
        parent.children.push(this);
    }

    this.filename = null;
    this.loaded = false;
    this.children = [];
}
```

定位到文件之后，NodeJs会新建一个模块对象，然后根据路径载入并编译。不同的文件拓展名，其载入的方法有所不同。
- `.js`文件，通过fs模块同步读取文件后编译执行。
- .`node`，这是C/C++编写的拓展文件，通过`dlopen()`方法加载最后编译生成的文件。
- `.json`，通过fs模块同步读取文件后，用`JSON.parse()`解析后返回结果。
- 其余扩展名文件，当作.js文件载入。

每一个编译成功的模块都会将其文件路径作为索引缓存在 `Module._cache` 对象上,以提高二次引入的性能。

**JavaScript模块的编译**

每个模块文件中存在着 `require` 、 `exports` 、 `module` 这3个变量,在编译的过程中,Node对获取的JavaScript文件内容进行了头尾包装。在头部添加了 `(function (exports, require, module, __filename, __dirname) {\n` ,在尾部添加了` \n});` 。一个正常的JavaScript文件会被包装成如下的样子:

```js
(function (exports, require, module, __filename, __dirname) {
    var math = require('math');
    exports.area = function (radius) {
      return Math.PI * radius * radius;
    };
});
```
这 样 每 个 模 块 文 件 之 间 都 进 行 了 作 用 域 隔 离 。 包 装 之 后 的 代 码 会 通 过 `vm` 原 生 模 块 的`runInThisContext ()` 方法执行(类似 `eval` ,只是具有明确上下文,不污染全局),返回一个具体的`function` 对象。

`exports` 对象是通过形参的方式传入的,直接赋值形参会改变形参的引用,但并不能改变作用域外的值。

```js
var change = function (a){
  a = 100;
  console.log(a);  // 100
}
var a = 10;
change(a);
console.log(a);  // 10
```

**C/C++模块的编译**

**JSON文件的编译**

NodeJs利用 `fs` 模块同步读取`JSON`文件的内容之后,调用 `JSON.parse ()` 方法得到对象,然后将它赋给模块对象的 `exports` ,以供外部调用。

### NodeJs核心模块

NodeJs的核心模块在编译成可执行文件的过程中被编译进了二进制文件。核心模块其实分为C/C++编写的和JavaScript编写的两部分,其中C/C++文件存放在Node项目的src目录下,JavaScript文件存放在lib目录下。

NodeJs的 `buffer` 、`crypto` 、 `evals` 、 `fs` 、 `os` 等模块都是部分通过C/C++编写的。


![依赖层次关系](http://upload-images.jianshu.io/upload_images/3248493-177b481df1694dcc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**JavaScript的一个典型弱点就是位运算。JavaScript的位运算参照Java的位运算实现,但是Java位运算是在 int 型数字的基础上进行的,而JavaScript中只有 double 型的数据类型,在进行位运算的过程中,需要将 double 型转换为 int 型,然后再进行。所以,在JavaScript层面上做位运算的效率不高。**

## 包与 NPM

CommonJS的包规范的定义其实也十分简单,它由包结构和包描述文件两个部分组成,前者用于组织包中的各种文件,后者则用于描述包的相关信息,以供外部读取分析。

![包组织模块示意图](http://upload-images.jianshu.io/upload_images/3248493-e2dd8a0f9932e25e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

包实际上是一个存档文件,即一个目录直接打包为.zip或tar.gz格式的文件,安装后解压还原为目录。完全符合CommonJS规范的包目录应该包含如下这些文件。

- package.json:包描述文件。
- bin:用于存放可执行二进制文件的目录。
- lib:用于存放JavaScript代码的目录。
- doc:用于存放文档的目录。
- test:用于存放单元测试用例的代码。

包描述文件用于表达非代码相关的信息,它是一个JSON格式的文件——package.json,位于包的根目录下,是包的重要组成部分。而NPM的所有行为都与包描述文件的字段息息相关。

对于NodeJs而言,NPM帮助完成了第三方模块的发布、安装和依赖等。借助NPM,Node与第三方模块之间形成了很好的一个生态系统。借助NPM,可以帮助用户快速安装和管理依赖包。

### NPM常用功能

**查看帮助**

- 在安装Node之后,执行 npm –v 命令可以查看当前NPM的版本


    npm -v


- 在不熟悉NPM的命令之前,可以直接执行NPM查看到帮助引导说明


    npm

- 安装依赖包是NPM最常见的用法,它的执行语句是


    npm install express

NPM会在**当前目录**下创建node_modules目录,然后在node_modules目录下创建express目录,接着将包解压到这个目录下。

- 进行全局模式安装


    npm install express –g

需要注意的是,全局模式并不是将一个模块包安装为一个全局包的意思,它**并不意味着可以从任何地方通过 `require()` 来引用到它**。实际上, `-g` 是将一个包安装为全局可用的可执行命令。它根据包描述文件中的 bin 字段配置,将实际脚本链接到与NodeJs可执行文件相同的路径下。

- 如果不能通过官方源安装,可以通过镜像源安装，在执行命令时,添加 `--registry=http://registry.url `即可


    npm install underscore --registry=http://registry.url

如果使用过程中几乎都采用镜像源安装,可以执行以下命令指定默认源

    npm config set registry http://registry.url


在执行 `npm uninstall <package> `时, `uninstall` 指向的脚本也许会
做一些清理工作等。

- 分析出当前路径下能够通过模块路径找到的所有包,并生成依赖树


    npm ls


# 异步I/O