---
title: (一)初识NodeJs
date: 2017-10-22
tags: [NodeJs, 阅读笔记]
---

> 说明：该学习笔记参考《深入浅出Node.js》在学习过程中，添加了自己的理解和适当的补充！仅供参考！

NodeJs的出现，让JavaScript工程师实现了独自完成全栈的梦想。NodeJs作为后端JavaScript的运行平台，保留了前端浏览器中那些熟悉的接口，没有改写语言本身的任何特性，依然基于**作用链**和**原型链**。

## NodeJs有以下几个特点：

### 异步I/O

异步I/O的最常见实现场景就是发起Ajax调用。这里演示的是一个Ajax请求：

<!--more-->

```js
$.post("/url", {"title": "这是参数"}， function(data){
    console.log("收到响应");
});
console.log("发送Ajax结束");
```

这里，我们会注意到，输出“发送Ajax结束”并不一定是在输出“收到响应”之后，因为“收到响应”的执行时间是不被预期的。这里是比较重要的异步的原则：‘Don`t call me, I will call you’，**注重结果，不关心过程**。

在NodeJs中，绝大多数的操作都是以异步的方式进行调用。在NodeJs中，我们可以从语言层面很自然的进行并行I/O操作，每个调用之间无需等待其他调用结束，在**编程模型上可以极大的提升效率**。

这里是两个文件读取任务的耗时取决于最慢的那个文件读取的耗时：

```js
var fs = require("fs");
fs.readFile("/path1", function(err, file){
    console.log("读取文件1完成");
});
fs.readFile("/path2", function(err, file){
    console.log("读取文件2完成");
});
```

### 事件与回调函数

NodeJs是将前端浏览器中广泛且成熟的事件引入后端，配合异步I/O，将事件点暴露给业务逻辑。

```js
var http = require("http");
var querystring = require("querystring");

// 侦听服务其的request事件

http.createServer(function(req, res){
  var postData = '';
  req.setEncoding('utf8');

  // 侦听请求的data事件

  req.on('data', function(chunk){
    postData += chunk;
  });

  // 侦听请求的end事件
  req.on('end', function(){
    res.end(postData);
  });

}).listen(8080);
console.log('server start at port:8080')
```
在web服务器绑定request事件，对于请求对象，为其绑定data事件和end事件。相应的在前端Ajax请求中绑定success事件，在发出请求后，只需关心请求成功时执行相应的业务逻辑即可。

```js
$.ajax({
  'url': '127.0.0.1:8080',
  'method': 'POST',
  'data': {},
  'success': function(data){
    console.log(data);
  }
})
```

以上代码只是演示，此处的ajax并不能运行。在此只是说明“事件”。

事件的编程方式具有**轻量级**、**松耦合**、**只关注事务点**等优势。但是也会造成在多个异步任务的场景下，事件和事件之间各自独立，如何协作是一个重要的问题。

回调函数无处不在。在JavaScript中，是**将函数作为第一等公民来对待的，可以将函数作为对象传递给方法作为参数进行调用**(这里说的是不是闭包？可以去深揪一下！)

回调函数是最好的接受异步调用、返回数据的方式，但是这种编程方式对于同步思路编程的人来说，是一大挑战。**代码的编写顺序与执行顺序并无关系**，在流程控制方面，由于穿插了异步和回调使得变得不是那么一目了然。但是对于业务的划分和对事件的提炼上复杂度与同步方式是一致的。

### 单线程

NodeJs保持了**JavaScript在浏览器中的单线程**的特点。而且在NodeJs中，JavaScript**与其他线程是无法共享任何状态的**。

单线程的最大好处就是：**不用向多线程那样处处在意状态的同步问题，没有死锁的的存在，也没有线程上下文交换（这点需要深入解释下）所带来的性能上的开销**。

当然单线程也是有很大的弱点，但是必须要积极面对才能享受到node带来的好处。主要有以下三大方面：

- 无法利用多核CPU。
- **错误会引起整个应用的退出，应用的健壮性值得考验**。
- **大量计算占用CPU导致无法继续调用异步I/O**。

在浏览器中**JavaScript与UI共用一个线程，JavaSript长时间执行会导致UI的渲染和相应被终端**(这里思考浏览器加载资源的方式是并行还是串行，如何提高这方面的性能！)

第三个弱点，有相应的解决方案，这里暂时先不提！

### 跨平台

兼容于Windows和Linux平台。

## 应用场景

### I/O密集型

NodeJs擅长I/O密集型的应用场景，面向网络且擅长并行I/O，能够有效的组织更多的硬件资源，

I/O密集的优势主要在于NodeJs利用**事件循环**的处理能力，而不是启动每一个线程为每一个请求服务，资源占用极少。

### 分布式应用

### 与遗留系统和平相处

### 是否不擅长CPU密集型业务


这里简单的认识NodeJs，了解了其特性和应用场景，接下来，需要深入了，加油哦！