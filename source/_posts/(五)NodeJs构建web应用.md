---
title: (五)NodeJs构建web应用
date: 2017-10-27
tags: [NodeJs, 阅读笔记]
---

> 说明：该学习笔记参考《Node.js开发指南》，但是选用的模板和书中不同，添加了自己的理解和适当的补充！仅供参考！

我们打算从零开始用Node.js实现一个微博系统,功能包括路由控制、页面模板、数据库访问、用户注册、登录、用户会话等内容。在这里我们会使用Express框架、MVC设计模式、Jkig模板和MongoDB数据库的操作。

## 构建项目

###1 Express 应用生成器

通过应用生成器工具 express 可以快速创建一个应用的骨架。执行一下命令进行安装到全局环境中

    npm install express-generator -g

<!-- more -->

![安装应用生成器.png](http://upload-images.jianshu.io/upload_images/3248493-3d62a95537360ce1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以看到同时安装了一些依赖。在当前目录下创建`blog`的应用。

    express blog

创建完成后进入应用。应用目录如下：

![目录结构](http://upload-images.jianshu.io/upload_images/3248493-8d3c92208166060f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后安装所有依赖包。

    npm install

启动项目。

（windows下）

    set DEBUG=blog & npm start

（Linux平台下）

    DEBUG=blog npm start

打开浏览器，访问`http://127.0.0.1:3000`即可看到应用。

## 选定swig模板引擎

express构建的应用默认使用ejs模板引擎，本人觉得这种奇怪的东东暂时难以接受，之前做过django的项目，最后选定swig，也比较看好它！

###1 安装swig

    npm install swig

###2 在app.js中配置如下：

```
var swig = require('swig');

var swig  = new swig.Swig();
app.engine('html', swig.readerFile);
app.set('view engine', 'html');
```

###3 模板编写

移除`views`下的`*.jade`，创建同名的`*.html`。

编写`index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>index</title>
</head>
<body>
    Welcome, {{ title }}
</body>
</html>
```

启动服务

    set DEBUG=blog &npm start

打开浏览器访问`http://127.0.0.1:3000`，能够看到：Welcome, Express

## 使用Bootstrap和界面设计

我们选定Bootstrap开始设计我们的界面。首先[下载](http://v3.bootcss.com/getting-started/#download)，解压之后，将文件夹改名为`bootstrap-dist`并放在`public/`中，同时下载最新的jquery，放在`public/javascripts/`中。

![添加bootstrap文件夹到项目中](http://upload-images.jianshu.io/upload_images/3248493-64988b292b7d5378.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里补充一下**Swig的使用**[参考](http://www.cnblogs.com/elementstorm/p/3142644.html)。

修改`layout.html`中的代码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}My Blog{% endblock %}</title>
    {% block head %}
        <link rel="stylesheet" href="/bootstrap-dist/css/bootstrap.min.css">
    {% endblock %}
</head>
<body>
    {% block content %}{% endblock %}
    <script src="/javascripts/jquery-3.2.1.js"></script>
    <script src="/bootstrap-dist/js/bootstrap.js"></script>
</body>
</html>
```

在`views/`下创建`login.html`，并编写代码如下：

```html
{% extends 'layout.html' %}

{% block title %}用户登录{% endblock %}

{% block content %}

    <div class="container" style="margin-top: 30px;">
        <div class="row clearfix">
            <div class="col-md-12 column">
                <form class="form-horizontal" role="form" method="post" action="/users/login">
                    <div class="form-group">
                         <label for="inputEmail3" class="col-sm-2 control-label">用户名</label>
                        <div class="col-sm-10">
                            <input type="text" class="form-control" id="inputText3" />
                        </div>
                    </div>
                    <div class="form-group">
                         <label for="inputPassword3" class="col-sm-2 control-label">密码</label>
                        <div class="col-sm-10">
                            <input type="password" class="form-control" id="inputPassword3" />
                        </div>
                    </div>
                    <div class="form-group">
                        <div class="col-sm-offset-2 col-sm-10">
                            <div class="checkbox">
                                 <label><input type="checkbox" />记住我</label>
                            </div>
                        </div>
                    </div>
                    <div class="form-group">
                        <div class="col-sm-offset-2 col-sm-10">
                             <button type="submit" class="btn btn-default">登陆</button>
                        </div>
                    </div>
                </form>
            </div>
        </div>
    </div>

{% endblock %}
```

目前登录的界面已经创建完成，接下来就是添加路由了。修改`routes/users.js`文件，添加`login` GET视图后文件中代码如下：

```js
var express = require('express');
var router = express.Router();

/* GET users listing. */
router.get('/', function(req, res, next) {
  res.send('respond with a resource');
});


router.get('/login', function(req, res){
    res.render('login');
})

module.exports = router;
```

重新启动服务，浏览器访问[http://127.0.0.1:3000/users/login](http://127.0.0.1:3000/users/login)，即可以看到登录输入框，则成功！


![登录界面](http://upload-images.jianshu.io/upload_images/3248493-ec03d6cd5bef8840.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 数据持久化——MongoDB

从官网[下载](https://www.mongodb.com/download-center#community)并安装。

### 连接数据库

打开工
程目录中的package.json，在`dependencies`属性中添加以下代码代码:

    "connect-mongo": ">= 0.1.7",
    "mongodb": ">= 0.9.9"

然后运行`npm install`更新依赖的模块。接下来在工程的目录中创建`settings.js`文件，这个文件用于保存数据库的连接信息。

将用到的数据库起名为`blog`：

```
module.exports = {
    cookieSecret: "blogid",
    db: "blog",
    host: "localhost",
    port: 27017
};
```

`db`是数据库的名称，`host`是数据库的地址，`port`是数据库的端口。`cookieSecret`用于`Cookie`加密与数据库无关，我们留作后用。

接下来在项目中创建`modules`目录，然后在其下中创建`db.js`：

```js
var settings = require("../settings");
var mongodb = require("mongodb");
var Db = mongodb.Db;
var Server = mongodb.Server;

module.exports = new Db(settings.db, new Server(settings.host, settings.port, {}));
```

以上代码通过`module.exports`输出了创建的数据库连接。

### 会话支持

打开`app.js`添加以下内容：

```js
var session = require('express-session');
var settings = require('./settings');
var MongoStore = require('connect-mongo')(session);

app.use(session({
    secret: settings.cookieSecret,
    store: new MongoStore({
        db: settings.db,
        url: 'mongodb://localhost/blog'  
    })
}));
```

其中`express.cookieParser()`是Cookie解析的中间件。`express.session()`则提供会话支持，设置它的`store`参数为`MongoStore`实例，把会话信息存储到数据库中，以避免丢失。

这里需要**注意的是在Express4.X版本中，session已经分离出来**，所以这里需要去手动下载：

    npm install express-session

还需要注意的是**store中的url是需要填上去的**！这些地方由于书中所用版本较低的原因，这些都是“坑”，就是改这些东西，和查资料折腾了一下午！

### 注册功能的实现

####1 注册界面

设计注册界面在`views`中添加`registered.html`：

```
{% extends 'layout.html' %}

{% block title %}用户注册{% endblock %}

{% block content %}

<div class="container" style="margin-top: 30px;width: 500px;">

    {% if success %}
    <div class="alert alert-success alert-dismissable">
        <button type="button" class="close" data-dismiss="alert" aria-hidden="true">×</button>
        <h4>
            {{success}}
        </h4>
    </div>

    {% endif %}
    {% if error %}
    <div class="alert alert-danger alert-dismissable">
        <button type="button" class="close" data-dismiss="alert" aria-hidden="true">×</button>
        <h4>
            {{error[0]}}
        </h4>
    </div>
    {% endif %}

    <div class="row clearfix">
        <div class="col-md-12 column">
            <form class="form-horizontal" role="form" method="post" action="/users/registered">
                <div class="form-group">
                    <label for="inputText3" class="col-sm-2 control-label">用户名</label>
                    <div class="col-sm-10">
                        <input type="text" class="form-control" id="inputText3" name="username"/>
                    </div>
                </div>
                <div class="form-group">
                    <label for="inputPassword3" class="col-sm-2 control-label">密码</label>
                    <div class="col-sm-10">
                        <input type="password" class="form-control" id="inputPassword3" name="password" />
                    </div>
                </div>
                <div class="form-group">
                    <div class="col-sm-offset-2 col-sm-10">
                        <div class="checkbox">
                            <label><input type="checkbox" />记住我</label>
                        </div>
                    </div>
                </div>
                <div class="form-group">
                    <div class="col-sm-offset-2 col-sm-10">
                        <button type="submit" class="btn btn-default">注册</button>
                    </div>
                </div>
            </form>
        </div>
    </div>
</div>

{% endblock %}
```

界面完成之后，添加路由，在users.js下添加以下内容：

```js
router.get('/registered', function(req, res){
    res.render('registered');
});
```

浏览器打开[http://127.0.0.1:3000/users/registered](http://127.0.0.1:3000/users/registered)，这个页面刚开始进去是这样子的：


![注册界面](http://upload-images.jianshu.io/upload_images/3248493-18c26bef7da0891c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当注册失败是这样子的：


![注册失败](http://upload-images.jianshu.io/upload_images/3248493-efba69537fd37231.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

成功我就不贴了。接下来继续：

####2 注册响应

在user.js中添加`registered`的`post`请求：

```js
var User = require('../modules/user')

router.post('/registered', function(req, res){
    if (!(req.body['username'])){
        req.flash("error", '用户名不能为空！');
        return res.redirect("/users/registered");
    }
    var user = new User({
        name: req.body['username'],
        password: req.body['password'],
    });

    // 检查用户是否存在
    User.get(user.name, function(err, has_user){
        if(has_user){
            err = { "errmsg": "用户已存在"};
        }
        if(err){
            req.flash("error", err.errmsg);
            return res.redirect("/users/registered");
        }

        // 保存新用户
        user.save(function(err){
            if(err){
                req.flash("error", err.errmsg);
                return res.redirect("/users/registered");
            }
            req.flash("success", "注册成功！");
            res.redirect("/users/login");
        });
    });
});
```

在这段代码中：

- **req.body**就是POST请求信息解析过后的对象。
- **req.flash**是Express提供的一个奇妙的工具，通过它保存的变量只会在用户当前和下一次的请求中被访问，之后会被清除。当然这里使用的时候需要就行一些设置，待会再说。
- **res.redirect**是重定向功能。
- **User**对象，是接下来要创建的用户模型。实现了用户的判断和保存等。

####3 创建用户模型

User是一个描述数据的对象，即MVC架构中的模型。在modules目录下创建user.js，编写内容如下：

```js
var mongodb = require('./db');

function User(user){
    this.name = user.name;
    this.password = user.password;
}

module.exports = User;

User.prototype.save = function save(callback){

    var user = {
        name: this.name,
        password: this.password
    };

    mongodb.open(function(err, db){
        if (err){
            return callback(err);
        }
        db.collection('users', function(err, collection){
            if(err){
                mongodb.close();
                return callback(err);
            }

            // 将name属性添加为索引
            collection.ensureIndex('name', {unique: true});
            // 写入新用户到文档
            collection.insert(user, function(err, user){
                mongodb.close();
                callback(err, user);
            });
        });

    });

};


User.get = function get(username, callback){

    mongodb.open(function(err, db){
        if (err){
            return callback(err);
        }

        db.collection('users', function(err, collection){
            if(err) {
                mongodb.close();
                return callback(err);
            }
            collection.findOne({name: username}, function(err, doc){

                // 对数据库操作完成之后，及时关闭数据库
                mongodb.close();

                // 如果同名的用户存在，那么就直接返回这个用户信息
                if(doc){
                    return callback(err, doc);
                }else{
                    return callback(err, null);
                }
            });

        });
    });
};
```

以上代码实现了两个接口，`User.prototype.save`和`User.get`，前者是**对象实例的方法**，用于将用户对象的数据保存到数据库中，后者是**对象构造函数的方法**，用于从数据库中查找指定的用户。

注意：以上两个接口的类型是不一样的，所以在前面`users.js`中，是使用`User.get(...)`和`var user = new User(..);user.save(...);`。

####4 视图交互

在视图中访问会话中的用户数据，同时为了显示错误和成功的信息，也要增加响应的函数。在app.js中添加以下内容：

```js
var flash = require('connect-flash');
app.use(flash());
app.use(function(req,res,next){
  res.locals.user=req.session.user;

  var err = req.flash('error');
  var success = req.flash('success');
  res.locals.error = err.length ? err : null;
  res.locals.success = success.length ? success : null;
   
  next();
});
```

`'connect-flash`也是需要通过`npm install connect-flash`进行安装的。这里还需要注意的是以上在app.js中添加的代码在文件中的顺序很重要。这点也是一个大坑：

数据库连接在相应函数前面，响应函数在路由前面，这样才能在代码中正确调用`flash()`方法。

####5 小段总结

到这里注册的功能差不多就完成了，假设没有出现意外的话。祝你我都好运，以上代码折腾了一天，但是基本上让自己了解了Express和MongoDB的相互配合使用，也对整个系统的数据流了解的更加透彻。

接下来就是实现登录和首页的问题了，到这里就不贴代码了，会将现在实现的这些代码放在github上，做一个tab，感兴趣的同学可以去看看。