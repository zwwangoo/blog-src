---
title: Javascript DOM阅读笔记
date: 2017-07-24
tags: [阅读笔记, JavaScript]
---

>之前在w3school上学习过js相关基本的东西，而且在项目开发中大量接触js和jquery，所以在阅读时，只是用本文记录一些平时不常用或者比较重要的知识点。


## 第二章

### 操作符 `==`和`===`

`===` 全等操作符，执行严格的比较，**不仅比较值，而且会比较变量的类型**

```js
var a = false;
var b = ""
if(a === b){
    alert("a === b");  // 不输出
}
if(a == b){
    alert("a == b");  // 输出
}

```

<!--more-->

### 变量的作用域

用var关键字明确的为函数变量设定作用域。

如果在某个函数中使用var，那个变量就将被视为一个 **局部**变量，它只存在于这个函数的上下文中；反之，如果没有var，那个变量将被视为一个 **全局** 变量，如果脚本里已经存在一个与之同名的全局变量，这个函数就会改变那个全局变量的值。

```js
function square(num){
    total = num * num;
    return total;
}
var total = 50;
var number = square(20);
alert(total);
```

### 对象

对象就是由一些属性和方法组合在一起而构成的一个数据实体。


## 第三章

### 获取元素

- **getElementById()** 返回 **一个** 给定Id属性元素节点对应的 **对象**
- **getElementsByTagName()** 返回一个对象数组
- **getElementsByClassName()**  返回一个对象数组


### 获取和设置属性

- **getAttribute()** 它只有一个参数，查询的属性的名字
- **setAttribute()** 对属性节点值做出修改

`objects.setAttribute(attribute, value)`


## 第四章


### 某给链接加onclick事件处理函数，并让这个处理函数所触发的JavaScript代码返回的值为true或false。

这样一来，当这个链接被点击时，如果JavaScript代码返回的值是true，onclick事件处理函数就认为“这个链接被点击了”；反之，如果返回的是false，onclick事件处理函数就认为“这个链接没有被点击”

    <a href="http://www.baidu.com" onclick="return false;">Click me</a>

当点击这个链接时，因为onclick事件处理函数所触发的Javascript代码返回给他的值是false，所以这个链接的默认行为没有被触发

    onclick = "showH(this);return false;"


## 第五章

### 最佳实践

- 平稳退化：确保网页在没有javascript的情况下也能正常工作
- 分离javascript：把网页的结构和内容与javascript脚本动作行为分开
- 向后兼容：确保老版本的浏览器不会因为你的js脚本而死掉
- 性能考虑： 确保脚本执行的性能最优

在实际开发过程中，很难保证以上各点。

### 压缩脚本

精简后的代码虽然不容易看懂，但能大幅减少文件大小。多数情况下，你应该有两个版本，一个是工作副本，可以修改代码并添加注释；另一个是精简副本，用户放在站点上。通常为了与非精简版本区分开，最好在精简副本的文件名上加上min字样。

代码压缩工具：
[UglifyJS](https://github.com/mishoo/UglifyJS2)


## 第七章

### innerHTML 读、写给定元素里的HTML内容

```html
<div id="testdiv">
</div>
```

在上述div中插入HTML

```javascript
window.onload = function(){
    var testdiv = document.getElementById("testdiv");
    testdiv.innerHTML = "<p> inserted <em>this</em> content.</p>";
}
```

利用这个技术，无法区分“插入一段HTML内容”和“替换HTML内容”。testdiv元素里有没有HTML内容无关紧要：一旦使用了innerHTML属性，它的全部内容都将被替换。


### DOM方法

- **createElement()** 
- **appendChild()**
- **createTextNode()** 
- **insertBefore()**

### AJAX

Ajax技术的核心就是XMLHttpRequest对象。

兼容所有浏览器的写法：

```js
function getHTTPObject(){
    if(typeof XMLHttpRequest == "undefined"){
        XMLHttpRequest = function(){
            try { return new ActiveXobject("Msxml2.XMLHTTP.6.0");}
                catch(e) {}
            try { return new ActiveXobject("Msxml2.XMLHTTP.3.0");}
                catch(e) {}
            try { return new ActiveXobject("Msxml2.XMLHTTP");}
                catch(e) {}
            return false;
        }
    }
    return new XMLHttpRequest();
}
```

#### open(method, url, async): 规定请求的类型、URL 以及是否异步处理请求。

- method：请求的类型；GET 或 POST
- url：文件在服务器上的位置
- async：true（异步）或 false（同步）

#### send(string): 将请求发送到服务器。

- string：仅用于 POST 请求


**异步请求有一个很容易被忽略的问题就是异步性，就是脚本在发送异步请求之后，仍会继续执行，不会等待相应！**

**Ajax的同源策略**


## 第九章

### 三位一体的网页

- 结构层：由HTML或XHTML之类的标记语言负责创建
- 表示层：由CSS负责完成
- 行为层：负责内容应该如何响应事件这一问题，这是js和DOM主宰的领域


### style 属性

styles属性包含着元素的样式，查询这个属性将返回一个对象而不是一个简单的字符串。样式都存放在这个style对象的属性里：

    element.style.property

当你需要引用一个中间带减号的css属性时，DOM要求你用驼峰命名法。例如，css属性 **font-family** 变成DOM属性 **fontFamily**


DOM style 属性不能用来检索在外部css文件中声明的样式，在外部样式表里声明的样式不会进入style对象，在文档的`<head>` 部分里声明的样式也是如此。


## 第十章

### `setTimeout()` 能够让某个函数在经过一段预定的时间之后才开始执行

    setTimeout("function", interval)


### Math常用函数

- `ceil(number)` 向上取整
- `floor(number)` 向下取整
- `round(number)` 将任意浮点数舍入为与之最相近的整数
