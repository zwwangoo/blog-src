---
title: 你不知道的javascript(中卷)阅读笔记
date: 2017-08-09
tags: [阅读笔记, JavaScript]
---


《你不知道的JavaScript》系列图书是很早之前一位学长推荐的，从买来之后，一直找各个理由往后拖。

自己虽然也能写一些js，但是，经过上次面试，才知道，自己的基础很差，当前是认真务实自己已经掌握的技术的基础。一步一个脚印，从头开始。

<!-- more -->

## 第一部分 类型和语法


### 第一章 类型

**有以下七种内置类型**

- 空值(null)
- 未定义(undefined)
- 布尔型(boolean)
- 数字(number)
- 字符串(string)
- 对象(object)
- 符号(symbol, ES6新增)

可以使用 `typeof` 运算符查看类型



**值和类型**

变量没有类型，但它们持有的值有类型。类型定义了值的行为。


typeof运算符总是返回一个字符串。


```js
/* 十进制转换成二进制 */
var a = 12;
console.log(a.toString(2));

/* 字符串转换成整数和浮点数注意的地方 */
var b = "123456red";
var c = "12.2";
var d = "123";
var e = "ddd";
console.log([parseInt(b), parseInt(c), parseInt(d), parseInt(e)]);
console.log([parseFloat(b), parseFloat(c), parseFloat(d), parseFloat(e)]);
```

### 第二章 值

**数组**

数组可以容纳任何类型的值，可以是字符串、数字、对象，甚至是其他数组。

对数组声明后即可向其中加入值，不需要预先设定大小。

```js
var a = [];
console.log(a.length);  // 0
a[0] = 1;
a[1] = "2";
a[3] = [3];
console.log(a.length); // 4
```

上述代码没有设置 `a[2]` 单元，那么 `a[2]` 的值为 **undefined**。

**字符串与数组一些比较**

字符串和数组很相似，它们都是类数组，都有 `length`属性以及 `indexOf()`和 `concat()`方法。

字符串是不可变的，而数组是可变的。

```js
/* 数组与字符串的对比 */
var f = "foo";
var g = ['f', 'o', 'o'];
var h = Array.prototype.join.call(f, "-");
var i = Array.prototype.map.call(f, function () {
    return "."
}).join("");
console.log({"h": h, "i": i});
console.log(g.reverse());  // 数组符串
```
数组有一个字符串没有的可变更成员的函数 `reverse()`

**简单字符串的反转实现：**

先将字符串转换成数组，待处理完再将结果转换回字符串

```js
/* 简单字符串反转 */
var sReverse = "reverse".split("").reverse().join("");
console.log(sReverse);
```

**数字**

`toFixed()` 方法可以指定小数部分的显示位数：

```js
var a = 42.49;
a.toFixed(0);  // 43
a.toFixed(1); // 42.5
a.toFixed(3); // 42.490
```
**小数点后小数部分最后面的0也可以省略。**

注意一下：

```js
// 无效语法
42.toFixed(3);

// 一下语法都有效

(42).toFixed(3);
0.42.toFixed(3);
42..toFixed(3);
42 .toFixed(3); // 42后有空格
```

`42.toFixed(3)`是无效语法，因为被视为常量42.0的一部分，所以没有 . 属性访问运算符来调用`toFixed()`


```js
console.log(0.1 + 0.2 === 0.3) // false
```

二进制浮点数中的0.1和0.2并不是十分准确，他们相加的结果并非刚好0.3。

**不是数字的数字**

检查参数是否不是 NaN，也不是数字

```js
/* 检查参数是否不是 NaN，也不是数字 */
if (!Number.isNaN){
    Number.isNaN = function (n) {
        return (
            typeof n === "number" &&
            window.isNaN(n)
        );
    };
}
console.log(isNaN(f));
```

NaN是javaScript中 **唯一**一个不等于自身的值。

**值与引用**

- 简单值(即标量基本类型值)总是通过值复制的方式赋值/传递，包括null、undefined、字符串、数字、布尔和ES6中的symbol。
- 复合值总是通过引用复制的方式来赋值/传递，包括对象（包括数组和封装对象）和函数。

由于引用指向的是值本身而非变量，所以一个引用无法更改另一个引用的人指向。

```js
var a = [1, 2, 3];
var b = a;
console.log({"a": a,"b": b}); // {"a": [1, 2, 3], "b": [1, 2, 3]}

b = [4, 5, 6];
console.log({"a": a,"b": b}); // {"a": [1, 2, 3], "b": [4, 5, 6]}
```

注意一下：

```js
function foo(x) {
    x.push(4);
    console.log(x); // [1, 2, 3, 4]

    x = [4, 5, 6]; // x赋值新的数组，所以对以前数组没有产生影响。
    x.push(7);
    console.log(x); // [4, 5, 6, 7]
}
var a = [1, 2, 3];
foo(a);
console.log(a);  // [1, 2, 3, 4]
```
区别：

```js
function foo(x) {
    x.push(4);
    console.log(x); // [1, 2, 3, 4]

    x.length = 0; // 清空数组，这里更改了a指向的数组的长度
    x.push(4, 5, 6, 7);
    console.log(x); // [4, 5, 6, 7]
}
var a = [1, 2, 3];
foo(a);
console.log(a);  // [4, 5, 6, 7]
```
从上例可以看出，`x.length = 0` 和 `x.push(4, 5, 6, 7)` 并没有创建一个新的数组，而是更改了当前的数组。


**补充**

所有变量在赋值之前默认值都是 `undefined`。void运算符返回undefined。

当JavaScript执行一个函数时，它首先会查找函数内所有的变量声明，它使用初始值 **undefined** 创建变量。

```js
var foo = 1;
function main() {
  console.log(foo);  // undefined
  var foo = 2;
  console.log(this.foo);  // 1
  this.foo = 3;
}
main();
```

### 第三章 原生函数

通过构造函数(如new String("abc"))创建出来的是封装了基本类型值的封装对象。

```js
var a = String("abc");
console.log(typeof a);  // 是"Object"， 不是 "String"
console.log(a instanceof String);  // true
Object.prototype.toString().call(a);  // "[object String]"
```

**封装对象包装**

由于基本类型值没有`.length`和`.toString()`这样的属性和方法，需要通过封装对象才能访问，
此时js会自动为基本类型值包装一个封装对象。
（基本类型能够使用`.length`, `.toString()`等属性和方法，是因为js自动为其包装一个封装对象。）

一般情况下，我们不需要直接使封装对象。

在需要用到封装对象中的基本类型值的地方会发生隐性**拆封**。

**原生函数作为构造函数**

- 关于数组、对象、函数和正则表达式，我们通常喜欢以常量的形式来创建它们。
- 应该尽量避免使用构造函数。
- Array构造函数只带一个数字参数的时候，该参数会被作为数组的预设长度，而非只充当数组中的一个元素。
- 永远**不要**创建和使用**空**单元数组。
- 强烈建议使用常量形式来定义正则表达式，这样不仅语法简单，执行效率也更高。因为js引擎在代码执行前会对他们进行预编译和缓存。
- 创建日期对象必须使用`new Date()`。
- 错误对象`Error()`通常与throw一起使用。

**常用的字符串对象方法**

- indexof() 在字符串中找到指定子字符串的位置
- chartAt() 获得字符串指定位置上的字符
- substr()、substring()、slice() 获取字符串的指定部分
- toUpperCase()、toLowerCase() 将字符串转换为大写或小写
- trim() 去掉字符串前后的空格，返回新的字符串


### 第四章 强制类型转换

**抽象值操作**

**toString**

负责处理非字符串到字符串的强制类型转换。

- 基本类型值： `null` 转换为`"null"`，undefined转换为`"undefined"`，`true`转换为`"true"`
- 普通对象：如果对象有自己的`toString()`方法，字符串化时就会调用该方法并返回其值，否则返回内部属相`[[Class]]`的值。

```js
var a = [1, 2, 3];
console.log(a); // "1, 2, 3"
```

**JSON字符串化**

工具函数 `JSON.stringify(..)` 在将JSON对象序列化为字符串时也用到了toString，但JSON字符串并非严格意义上的强制类型转换。对大多数简单值来说，效果基本相同。

```js
JSON.stringify(45); // "45"
JSON.stringify(null);  // "null"
```

`JSON.stringify()`在对象中遇到**undefined**、**function**、**symbol**时会自动将其忽略，在数组中则会返回**null**

```js
JSON.stringify(undefined);  // undefined
JSON.stringify(function(){});  // undefined
JSON.stringify([1, undefined, function(){}, 4]); // "[1, null, null, 4]"
```

对包含循环引用的对象执行JSON.stringify()会出错。

我们可以向`JSON.stringify()`传递一个可选参数replacer，它可以是数组或者函数，用来指定对象序列化过程中哪些属性则应给被处理，哪些应该被排除。

```js
var a = {
    b: 45,
    c: "45",
    d: [1, 2, 3]
};
JSON.stringify(a, ["b", "c"]);  // "{"b":45,"c":"45"}"
```

- 字符串、数字、布尔值和null的`JSON.stringify()`规则与toString基本相同。
- 如果传递给`JSON.stringity()`的对象包含了`toJSON()`方法，那么该方法会在字符串化前调用，以便将对象转换成安全的JSON值


**ToNumber**

true转换成1，false转换为0.undefined转换为NaN，null转换为0。

处理失败时返回 **NAN** (处理数字常量失败时会产生错误语法)；对以0开头的十六位进制数不按照十六位进制处理。而是按照十进制。

对于对象：首先检查该值是否有`valueOf()`方法，如果有并且返回基本类型值，就使用该值进行强制类型转换。如果没有就是用`toString()`的返回值(如果存在)来进行强制类型转换。

```js
var a = {
    valueOf: function() {
      return "42";
    }
};

var b = {
    toString: function() {
      return "42";
    }
};

var c = [4, 2];
c.toString = function() {
  return this.join("");
};

Number(a);  // 42
Number(b);  // 42
Number(c);  // 42
Number([]);  // 0
Number(["abc"]);  // NaN  
```

**ToBoolean**

假值：

- undefined
- null
- false
- +0、-0和NaN
- ""

假值的布尔类型转换结果为false,假值列表以外的值都是真值。


**显式强制类型转换**

字符串与数字之间的显式转换

```js
var c = "3.14";
var d = +c;
console.log(typeof c);  // string
console.log(typeof d);  // number
```

上例中`+c` 是 **+** 运算符的一元形式。**+** 运算符显式的将c转换为数字，而非数字加法运算；另一个常见的用途就是将日期对象强制转换为数字，返回结果为Unix时间戳，以毫秒为单位（从1970年1月1日00:00:00UTC到当前时间），但是不建议使用。

**隐式强制类型转换**

隐式强制类型转换的作用是减少冗余，让代码更简洁。

如果+的其中一个操作数是字符串（或者是通过一些步骤可以得到字符串），则执行字符串拼接，否则执行数字加法。

我们可以将数字和空字符串`""`相+将其转换为字符串。

```js
var a = 42;
var b = a + "";
console.log(b); // "42"
```
下列情况会发生**布尔值隐式强制类型转换**：

- **`if()`**语句中的条件判断表达式。
- **`for(..; ..; ..;)`**语句中的条件判断表达式。
- **`while(..)`**和**`do..while(..)`**循环中的条件表达式。
- **`? : `**中的条件判读表达式。
- 逻辑运算符 **`||`** 和 **`&&`** 左边的操作数。


**`||`和`&&`**


在JavaScript中它们返回的并不一定是布尔类型，而是两个操作数其中一个的值。

```js
var a = 42;
var b = 'abc';
var c = null;

console.log(a || b);  // 42
console.log(a && b);  // 'abc'
console.log(c || b);  // 'abc'
console.log(c && b);  // null
```
**`||`** 和 **`&&`** 首先对第一个操作数执行条件判断。

- 对于 **`||`** 来说， 如果条件判断结果为 `true` 就返回第一个操作数的值，否则就返回第二个操作数的值。
- 对于 **`&&`** 则相反，如果条件判断结果为 `true`就返回第二个操作数的值，否则就返回第一个操作数的值。

换一个角度理解：

`a || b` 相当于 `a ? a : b`， 注意的是a只执行一次。

`a && b` 相当于 `a ? b : a`

```js
function foo(a, b) {
    a = a || "hello";
    b = b || "world";
    console.log(a + " " + b);
}
foo();  // "hello world"
foo("yeah", "yeah");  // "yeah yeah"
```

**宽松相等和严格相等**

**`==`允许在相等比较中进行强制类型转换，而`===`不允许**

如果两个值的类型不同，我们就需要考虑有没有强制类型转换的必要，有就用 `==`，没有就用`===`，不用在乎性能。

容易出错的一个地方：

```js
var a = "42";
var b = true;
var c = false;

console.log(a == b);  // false
console.log(a == c);  // false
```

- 如果Type(x)是布尔类型，则返回`ToNumber(x) == y`的结果。
- 如果Type(y)是布尔类型，则返回`x == ToNumber(y)`的结果。

**避免使用 `== true` 和`== false`**

```js
var a = "42";
/* 建议使用一下显示 */
if(a){
    //  ..
}

if(!!a){
    //  ..
}

if(Boolean(a)){
    // ..
}
```

在 `==` 中 **null** 和 **undefined** 相等，除此之外其他值都不存在这种情况。

注意：

```js
var i = 2;
Number.prototype.valueOf = function() {
  return i++ ;
};
var a = new Number(42);
if (a == 2 && a == 3){
    console.log("happened");  // "happened"
}
```
以下

```js
var i = 2;
Number.prototype.valueOf = function() {
  return i++ ;
};
var a = Number(42);
if (a == 2 && a == 3){
    console.log("happened");
}
```

几种特殊情况：

```
"" == 0;  // true
"" == []; // true
0 == [];  // true
```
这几种特殊情况，在实际开发中会不小心使用上，特别是 `== ""`和 `== 0`，应当注意。

以下两个原则：

- 如果两边的值中有`true`或者`false`，千万不要使用 `==`。
- 如果两边的值中有`[]`、`""`或者`0`，尽量不要用`==`。

这时最好用`===`

在js中 `<=`是**不大于**的意思(即`!(a > b)`，处理`!(b < a)`)。同理`a>= b`处理为`b <= a`。

```
var a = {b: 42};
var b = {b: 43};

a < b;  // false
a == b;  // false
a > b;  //false

a <= b;  // true
a >= b;  // true
```

### 第五章 语法

语句都有一个结果值(undefined也算)

代码块`{..}`的结果是最后一个语句/表达式的结果。换句话说，代码块的返回值就如同**一个隐式的返回**，即返回最后一个语句的结果值。

```js
var a = 42;
var b = (a++);

console.log(a);  // 43
console.log(b);  // 42
```
有人误以为可以用括号`()`将`a++`的副作用封装起来，例如上例。但是并非如此，`()`本身并不是一个封装表达式，不会在表达式a++产生副作用之后执行，即便如此a++首先返回42。

**链式赋值**

```js
var a, b , c;
a = b = c = 42;
var d = e = 42;
```

a， b，c，d，e均被赋值为42。变量 e,没有在作用域中像`var e` 这样声明过，则`var d = e = 42`不会对变量e进行声明。

JavaScript通过标签跳转能够实现goto的部分功能。`continue`和`break`语句都可以带一个标签。

```js
foo: for(var i=0; i<4; i++){
    for(var j=0; j<4; j++){
        if (j == i){
            continue foo;  // 跳到foo的下一个循环
        }
        if ((j * i) % 2 == 1){
            continue;  //继续内层循环
        }

        console.log(i, j)
    }
}
```
注意上例：`contiue foo` 并不是指“跳转到标签foo所在位置继续执行”，而是“执行foo循环的下一轮循环”，所以这里的foo并非goto。

带标签的循环跳转一个更大的用处在于，和`break _` 一起使用可以实现从内部循环跳转到外层循环。

```js
foo: for(var i=0; i<4; i++){
    for(var j=0; j<4; j++){
        if(i*j >= 3){
            console.log("stopping!", i, j);
            break foo;
        }
        console.log(i, j);
    }
}
```
注意上例：`break foo` 并不是指“跳转到标签foo所在位置继续执行”，而是“跳出标签foo所在的循环/代码块，继续执行后面的代码”，所以这里的foo并非goto。

标签也可以用于非循环代码块，但是只有`break`才可以。我们可以对带标签的代码块使用`break __`，但是不能对带标签的非循环代码块使用`continue __`， 也不能对不带标签的代码块使用`break`。

标签不允许使用双引号。

**运算符优先级**

- 用 `,`来连接连接一系列语句的时候，它的优先级最低，其他操作数的优先级都比它高。
- `&&` 运算符优先于 `||`。而 `||`的优先级又高于 `? :`。
- `? :`是右关联，而且它的组合方式会影响返回结果。`=`也是右关联。

`do..while`循环后面必须带**`;`**，而 `while`和`for`循环后则不需要。

**try..finally**

`finally`中的代码总会在`try`之后执行，如果有`catch`的话则在`catch`之后执行。即无论出现什么情况最后一定会被调用。

```js
function foo(){
    try {
        return 42;
    }
    finally {
        console.log("hello");
    }
    console.log("nerver runs");
}

foo();
// hello
// 42
```

如果`finally`中抛出异常（无论是有意还是无意），函数就会在此终止。如果此前`try`中已有`return`设置返回值，则该值会被丢弃。

```js
function foo() {
    try {
        return 42;
    }
    finally {
       throw "Oops!";
    }
    console.log("nerver run");
}

foo();  // Uncaught Oops!
```

- `finally`中的`return`会覆盖`try`和`catch`中`return`的返回值。
- switch表达式的匹配算法与 `===`相同。
- 尽量不要是用`arguments`，如果非用不可，也切勿同时使用`arguments`和其对应的命名参数。

---
