---
title: (四)NodeJs核心模块
date: 2017-10-26
tags: [NodeJs, 阅读笔记]
---


> 本学习笔记是根据《Node.js开发指南》一书进行学习。

## 全局对象

JavaScript中有一个特殊的对象，称为全局对象（Global Object），**它及其所有属性都可以在程序的任何地方访问**，即全局变量。在浏览器JavaScript中，通常window是全局对象，而NodeJs中的全局对象是global，所有全局变量（除了global本身以外）都是global对象的属性。

### 全局对象和全局变量

按照ECMAScript的定义，满足以下条件的变量是全局变量：

- 在最外层定义的变量
- 全局对象的属性
- 隐式定义的变量（未定义直接赋值的变量）

在NodeJs中你不可能在最外层定义变量，因为所有用户代码都是属于当前模块的，而模块本身不是最外层上下文。

<!-- more -->

**永远使用var定义变量以避免引入全局变量，因为全局变量会污染命名空间，提高代码的耦合风险。**

### process

process是一个全局变量，即global对象的属性。它用于描述当前NodeJs进程状态的对象，提供了一个与操作系统的简单接口。

- `process.argv`是命令行参数数组，第一个元素是node，第二个元素是脚本文件名，从第三个元素开始每个元素是一个运行参数。
- `process.stdout`是标准输出流，通常我们使用的console.log()向标准输出打印字符，而`process.stdout.write()`函数提供了更底层的接口。
-` process.stdin`是标准输入流，初始时它是被暂停的，要想从标准输入读取数据，你必须恢复流，并手动编写流的事件响应函数。
- `process.nextTick(callback)`的功能是为事件循环设置一项任务，NodeJs会在下次事件循环调响应时调用`callback`。

```js
// debug.js
console.log(process.argv);

process.stdin.resume();
process.stdin.on("data", function (data) {
    process.stdout.write(data.toString());
});
```

结果：

![image.png](http://upload-images.jianshu.io/upload_images/3248493-c706e1b494bc5f76.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**不要使用`setTimeout(fn, 0)`代替`process.nextTick(callback)`，前者比后者效率要低得多。**

### console

`console`对象用于向标准输出流（stdout）或标准错误流（stderr）输出字符。

- console.log()向标准输出流打印字符并以换行符结束。
- console.error():与console.log()用法相同，只是向标准错误流输出。
- console.trace():向标准错误流输出当前的调用栈。

console.log接受若干个参数，如果只有一个参数，则输出这个参数的字符串形式。如果有多个参数，则以类似于C语言printf()命令的格式输出。第一个参数是一个字符串，如果没有参数，只打印一个换行。

## 常用工具util

`util`是一个Node.js核心模块，提供常用函数的集合

### util.inherits

**JavaScript的面向对象特性是基于原型的，与常见的基于类的不同。JavaScript没有提供对象继承的语言级别特性，而是通过原型复制来实现的**

`util.inherits(constructor, superConstructor)`是一个实现对象间原型继承的函数。

```js
// inherits.js
var util = require("util");

function Base() {
    this.name = "base";
    this.base = 1991;
    this.sayHello = function () {
        console.log("Hello"+this.name);
    };
}
Base.prototype.showName = function () {
    console.log(this.name);
};

function Sub() {
    this.name = "sub";
}

util.inherits(Sub, Base);

var newBase = new Base();
newBase.showName();
newBase.sayHello();
console.log(newBase);

var newSub = new Sub();
newSub.showName();
// newSub.sayHello();
console.log(newSub);
```

![image.png](http://upload-images.jianshu.io/upload_images/3248493-c100f35b4e1e1887.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`Sub`仅仅继承了`Base`在原型中定义的函数，而构造函数内部创造的`base`属性和`sayHello`函数都没有被Sub继承。同时，在原型中定义的属性不会被console.log作为对象的属性输出。

### util.inspect

`util.inspect(object,[showHidden],[depth],[colors])`是一个将任意对象转换为字符串的方法，通常用于调试和错误输出。它至少接受一个参数object，即要转换的对象。

`util`还提供了`util.isArray()`、`util.isRegExp()`、`util.isDate()`、`util.isError()`四个类型测试工具，以及`util.format()`、`util.debug()`等工具。

## 事件驱动events

events是NodeJs最重要的模块。NodeJs本身架构就是事件式的，而它提供了唯一的接口，所以堪称NodeJs事件编程的基石。

### 事件发射器

`events`模块只提供了一个对象：`events.EventEmitter`。`EventEmitter`的核心就是**事件发射与事件监听器功能的封装**。`EventEmitter`的**每个事件由一个事件名和若干个参数组成**，**事件名是一个字符串**，通常表达一定的语义。对于每个事件，`EventEmitter`支持若干个事件监听器。当事件发射时，注册到这个事件的事件监听器被依次调用，事件参数作为回调函数参数传递。

```js
var EventEmitter = require("events").EventEmitter;
var event = new EventEmitter();
event.on("some_event", function(){
    console.log("some_event start1");
});
event.on("some_event", function () {
    console.log("some_event start2")
});

event.emit("some_event");
```

执行以上代码输出：
```
some_event start1
some_event start2
```

运行结果中可以看到两个事件监听器回调函数被先后调用。

`EventEmitter`常用的API：

- `EventEmitter.on(event, listener)`为指定事件注册一个监听器，接受一个字符串`event`和一个回调函数`listener`。
- `EventEmitter.emit(event, [arg1], [arg2], [...])`发射`event`事件，传递若干可选参数到事件监听器的参数表。
- `EventEmitter.once(event, listener)`为指定事件注册一个单次监听器，即监听器最多只会触发一次，触发后立刻解除该监听
- `EventEmitter.removeListener(event, listener)`移除指定事件的某个监听器，`listener`必须是该事件已经注册过的监听器。
- `EventEmitter.removeAllListeners([event])`移除所有事件的所有监听器，如果指定`event`，则移除指定事件的所有监听器。

```js
var EventEmitter = require("events").EventEmitter;
var event = new EventEmitter();
event.on("some_event", function(){
    console.log("some_event start1");
});
event.on("some_event", function () {
    console.log("some_event start2")
});

event.emit("some_event");

setTimeout(function(){
    event.emit("some_event");
}, 1000);

event.removeAllListeners("some_event");
event.emit("some_event");
```
执行以上代码，输出：

![image.png](http://upload-images.jianshu.io/upload_images/3248493-467bc54781ec87df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### error事件

`EventEmitter`定义了一个特殊的事件`error`，它包含了“错误”的语义，我们在遇到异常的时候通常会发射`error`事件。

### 继承EventEmitter

大多数时候我们不会直接使用EventEmitter，而是**在对象中继承它**。包括`fs`、`net`、`http`在内的，只要是支持事件响应的核心模块都是`EventEmitter`的子类。

原因有两点。首先，具有某个实体功能的对象实现事件符合语义，事件的监听和发射应该是一个对象的方法。其次JavaScript的对象机制是基于原型的，支持部分多重继承，继承`EventEmitter`不会打乱对象原有的继承关系。

## 文件系统fs

### fs.readFile

`fs.readFile(filename, [encoding], [callback(err, data)])`是最简单的读取文件的函数。

```js
var fs = require("fs");
fs.readFile("server.js", "utf-8", function(err, data){
    if (err){
        console.log(err);
    }else{
        console.log(data);
    }
})
```

### fs.readFileSync

`fs.readFileSync(filename, [encoding])`是`fs.readFile`同步的版本。它接受的参数和`fs.readFile`相同，而读取到的文件内容会以函数返回值的形式返回。如果有错误发生，fs将会抛出异常，你需要使用`try`和`catch`捕捉并处理异常。

### fs.open

### fs.read

一般来说，除非必要，否则不要使用以上两种方式读取文件，因为它要求你手动管理缓冲区和文件指针，尤其是在你不知道文件大小的时候，这将会是一件很麻烦的事情。

## HTTP服务器与客户端

### HTTP服务器

**http.Server的事件**

`http.Server`是一个基于事件的HTTP服务器,所有的请求都被封装为独立的事件,开发者只需要对它的事件编写响应函数即可实现HTTP服务器的所有功能。它继承自`EventEmitter`,提供了以下几个事件：

- request:当客户端请求到来时,该事件被触发,提供两个参数`req`和`res`,分别是`http.ServerRequest`和`http.ServerResponse`的实例,表示请求和响应信息。
- connection:当TCP连接建立时,该事件被触发,提供一个参数`socket`,为`net.Socket`的实例。`connection`事件的粒度要大于`request`,因为客户端在Keep-Alive模式下可能会在同一个连接内发送多次请求。
- close :当服务器关闭时,该事件被触发。注意不是在用户连接断开时。
- checkContinue、upgrade、clientError事件。

最常用的就是`request`了,因此`http`提供了一个捷径:`http.createServer([requestListener])`,功能是创建一个HTTP服务器并将`requestListener`作为`request`事件的监听函数。

**http.ServerRequest**

一般由`http.Server`的`request`事件发送,作为第一个参数传递,通常简称request或req。`http.ServerRequest`提供了以下3个事件用于控制请求体传输：

- data :当请求体数据到来时,该事件被触发。该事件提供一个参数chunk,表示接收到的数据。如果该事件没有被监听,那么请求体将会被抛弃。该事件可能会被调用多次。
- end :当请求体数据传输完成时,该事件被触发,此后将不会再有数据到来。
- close:用户当前请求结束时,该事件被触发。不同于end,如果用户强制终止了传输,也还是调用close。

**获取GET请求内容**

`url`模块中的parse函数提供解析客户端的表单请求。

```js
// httpServerRequestGet.js
var http = require("http");
var url = require("url");
var util = require("util");

http.createServer(function(req, res){
    res.writeHead(200, {"Conetnet-Type": "text/html"});
    res.end(util.inspect(url.parse(req.url, true)));
}).listen(3000);
```

在浏览器中访问`http://127.0.0.1:3000/user?name=byvoid&email=byvoid@byvoid.com`

![image.jpg](http://upload-images.jianshu.io/upload_images/3248493-d9e60b84f5cb9090.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过`url.parse`,原始的`path`被解析为一个对象,其中`query`就是我们所谓的GET请求的内容,而路径则是`pathname`。

**获取POST请求内容**

```js
// httpServerRequestPost.js
var http = require("http");
var querystring = require("querystring");
var util = require("util");

http.createServer(function(req, res){
    var post = "";
    req.on("data", function(chunk){
        post += chunk
    });
    req.on("end", function(){
        post = querystring.parse(post);
        res.end(util.inspect(post));
    });
}).listen(3000)
```

通过事件监听函数。上面的代码仅供理解使用，在实际编码中不赞同这样的做法。

**http.ServerResponse**

返回给客户端的信息，也是由http.Server的request事件发送的,作为第二个参数传递,一般简称为response或res。

http.ServerResponse有三个重要的成员函数,用于返回响应头、响应内容以及结束请求。

- response.writeHead(statusCode, [headers])：该函数在一个请求内最多只能调用一次。
- response.write(data, [encoding])：在response.end调用之前,response.write可以被多次调用。
- response.end([data], [encoding])： 结束响应,告知客户端所有发送已经完成。

### HTTP客户端

http模块提供了两个函数`http.request`和`http.get`。

- http.request(options,callback)发起HTTP请求。接受两个参数,`option`是一个类似关联数组的对象,表示请求的参数,`callback`是请求的回调函数。

```js
// httpRequst.js
var http = require("http");
var querystring = require("querystring");

var content = querystring.stringify({
    name: "byvoid",
    email: "byvoid@byvoid.com",
    address: "Tangshan",
});

var options = {
    host: "127.0.0.1",
    port: 3000,
    path: '/user?hello=wen',
    method: "POST",
    headers: {
        "Content-Type": "application/x-www-form-urlencoded",
        "Content-Length": content.length
    }
};

var req = http.request(options, function(res){
    res.setEncoding("utf-8");
    res.on("data", function(data){
        console.log(data);
    });
});

req.write(content);
req.end();
```

- http.get(options, callback) `http`模块还提供了一个更加简便的方法用于处理GET请求:`http.get`。它是`http.request`的简化版,唯一的区别在于`http.get`自动将请求方法设为了GET请求,同时不需要手动调用`req.end()`。

**http.ClientRequest**

`http.ClientRequest`是由`http.request`或`http.get`返回产生的对象,表示一个已经产生而且正在进行中的HTTP请求

**http.ClientResponse**

提供了三个事件`data`、`end`和`close`,分别在数据到达、传输结束和连接结束时触发,其中`data`事件传递一个参数`chunk`,表示接收到的数据。