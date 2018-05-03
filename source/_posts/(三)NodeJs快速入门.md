---
title: (三)NodeJs快速入门
date: 2017-10-25
tags: [NodeJs, 阅读笔记]
---

> 本学习笔记是根据《Node.js开发指南》一书进行学习。前面的几篇是根据《深入浅出Node.js》学习，但是学习到第三四章关于异步I/O和异步编程时，就暂时先放弃了，主要因为那本书讲的有点深，更多从底层说起，对于从来没有接触过NodeJs的同学来说，学习起来有一些难度。《Node.js开发指南》这本书比较适合新手学习。当然本人也算新手。

## 开始用NodeJs编程

### Hello World 

创建`hello.js`编写如下代码：

    conlose.log("Hello World")

打开终端，进入`hello.js`所在目录，执行命令：

    node hello.js

`console.log`是我们最常用的输出指令,它和C语言中的`printf`的功能类似,也可以接受任意多个参数,支持`%d`、`%s`变量引用。

    console.log("%d", 1000)

<!-- more -->

### 建立HTTP服务器

Node.js将“HTTP服务器”这一层抽离,直接面向浏览器用户。


![Node.js与PHP的架构](http://upload-images.jianshu.io/upload_images/3248493-a60384e1e638e02d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

让我们创建一个HTTP服务，建立一个名为app.js的文件

```js
// app.js
var http = require("http");

http.createServer(function(req, res){
    res.writeHead(200, {"Content-Type": "text/html"});
    res.write("<h1>Hello world!</h1>");
    res.end("<p>NodeJs</p>");
}).listen(3000);

console.log("Server start at port 3000!")

```

运行`node app.js`命令，打开浏览器访问http://127.0.0.1:3000，即可看到输出。

`listen`函数中创建了事件监听器，使得Node.js进程不会退出事件循环。

## 异步式I/O与事件式编程

NodeJs最大的特点就是异步式I/O（或者非阻塞I/O）与事件紧密结合的编程模式。控制流很大程度上要靠事件和回调函数来组织，一个逻辑要拆分为若干个单元。

### 阻塞与线程

### 回调函数

在NodeJs中如何用异步的方式读取一个文件，下面是一个例子：

```js
// readFile.js
var fs = require("fs");

fs.readFile("server.js", "utf-8", function (err, data) {
    if(err){
        console.log("file read err!");
    }else {
        console.log(data);
    }
});
console.log("End!")
```

运行文件会发现，首先输出“End!”，然后才是读取文件的内容！

NodeJs也支持同步读取文件：

```js
//readFile.js

var fs = require("fs");
var data = fs.readFileSync("server.js", "utf-8");
console.log(data);
console.log("End!");
```

运行文件，先输出文件内容，最后输出"End!"

异步式I/O是通过回调函数来实现的。fs.readFile接收了三个参数，第一个是文件名，第二个是编码方式，第三个是一个函数，我们称这个函数为回调函数。

**NodeJs中，并不是所有的API都提供了同步和异步版本。Node.js不鼓励使用同步I/O。**

### 事件
NodeJs所有的异步I/O操作在完成时都会发送一个事件到事件队列。事件由`EventEmitter`对象提供

```js
//event.js
var EventEmitter = require("events").EventEmitter;
var event = new EventEmitter();
event.on("some_event", function(){
    console.log("some_event occured");
});
setTimeout(function(){
    event.emit("some_event");
}, 1000);
```

运行这段代码，1秒后控制台输出了`some_event occured`。其原理是`event`对象注册了事件`some_event`的一个监听器，然后我们通过`setTimeout`在1000毫秒以后向`event`对象发送事件`some_event`，此时会调用`some_event`的监听器。

NodeJs程序由事件循环开始，到事件循环结束，所有的逻辑都是事件的回调函数，所以NodeJs始终在事件循环中，程序入口就是事件循环第一个事件的回调函数。事件的回调函数在执行的过程中，可能会发出I/O请求或直接发射（emit）事件，执行完毕后再返回事件循环，事件循环会检查事件队列中有没有未处理的事件，直到程序结束。

### 模块和包

NodeJs提供了require函数来调用其他模块，而且模块都是基于文件的，机制十分简单。

NodeJs提供了`exports`和`require`两个对象，其中`exports`是模块公开的接口，`require`用于从外部获取一个模块的接口，即所获取模块的`exports`对象。

让我们以一个例子来了解模块。

```js
// module.js
var name;
exports.setName = function(thyName){
    name = thyName;
}
exports.getName = function(){
    console.log("Hello " + name);
}
```

编写getModule.js

```
// getModule.js
var myModule = require("./module");
myModule.setName("Hak");
myModule.getName();
```

运行`getModule.js`会输出： `Hello Hak`

**单次加载**

上面这个例子有点类似于创建一个对象，但实际上和对象又有本质的区别，因为require不会重复加载模块，也就是说无论调用多少次require，获得的模块都是同一个。

我们在getModule.js的基础上稍作修改：

```js
var myModule1 = require("./module");
myModule1.setName("Hak");

var myModule2 = require("./module");
myModule2.setName("Jack");

myModule1.getName();
```
运行后发现输出结果是： Hello Jack。这是因为myModule1 和myModule2指向的是同一个实例，前者的结果被后者覆盖。

事实上，exports本身仅仅是一个普通的空对象，即{}，它专门用来声明接口，本质上是通过它为模块闭包
的内部建立了一个有限的访问接口。因为它没有任何特殊的地方，所以可以用其他东西来代替。

**不可以通过对exports直接赋值代替对module.exports赋值。exports实际上只是一个和module.exports指向同一个对象的变量，它本身会在模块执行结束后释放，但module不会，因此只能通过指定module.exports来改变访问接口。**

### 模块和包

npm是NodeJs官方提供的包管理工具。npm提供了命令行工具，使你可以方便地下载、安装、升级、删除包，也可以让你作为开发者发布并维护包。

在使用npm安装包的时候，有两种模式：本地模式和全局模式。默认情况下我们使用npm install命令就是采用本地模式，即把包安装到当前目录的node_modules子目录下。

npm还有另一种不同的安装模式被成为全局模式，使用方法为：

     npm install -g [page_name]

|模式|可通过require使用|注册PATH|
|--|--|--|
|本地模式|是|否|
|全局模式|否|是|


当我们要把某个包作为工程运行时的一部分时，通过本地模式获取，如果要在命令行下使用，则使用全局模式安装。

## 调试

### 命令行调试

NodeJs支持命令行下的单步调试。

```js
// debug.js
var a = 1;
var b = 'world';
var c =function(x){
    console.log('hello' + x + a);
};
c(b);
```

在命令行下执行`node debug debug.js`，将会启动调试工具：

![image.png](http://upload-images.jianshu.io/upload_images/3248493-5a5a44a085d529dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样就打开了一个NodeJs的调试终端。可以用一些基本的命令进行单步跟踪调试。

|命令|功能|
|-|-|
|run|执行脚本在第一行暂停|
|restart|重新执行脚本|
|cont, c|继续执行，直到遇到下一个断点|
|next, n|单步执行|
|step, s|单步执行并进入函数|
|kill|终止当前执行的脚本|
|...|...|

### 远程调试

V8提供的调试功能是基于TCP协议的，因此Node.js可以轻松地实现远程调试。在命令行下使用以下两个语句之一可以打开调试服务器。

    node --debug[=port] script.js
    node --debug-brk[=port] script.js

### 利用工具

### 使用node-inspector调试NodeJs