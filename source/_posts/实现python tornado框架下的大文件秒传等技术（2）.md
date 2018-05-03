---
title: 实现python tornado框架下的大文件秒传等技术（2）
date: 2017-07-28
tags: [tornado, python, 技术]
---

这一步实现了tornado基本的文件上传和保存，使用js的Ajax异步提交文件表单

## 前台界面

### css样式表

在 **static/css** 目录下创建样式表 **stylesheet.css**

<!--more-->

```css
* {
    font-family: "微软雅黑";
    margin: 0;
    padding: 0;
}

.container {
    padding-top: 10px;
    padding-left: 10px;
}

.container input {
    width: 120px;
    height: 30px;
    background-color: blue;
    color: white;
    border: 0;
    line-height: 30px;
    border-radius: 5px;
    margin-right: 5px;
    outline: none;
    cursor: pointer;
}

#filelist {
    width: 800px;
    border: solid 1px #eee;
    border-collapse: collapse;
    margin: 10px;
}

#filelist td {
    border-bottom: solid 1px #eee;
    height: 30px;
    font-size: 12px;
    /*line-height:30px ;*/
    padding: 0 3px;
}

.filename {
    width: 200px;
    text-align: center;
}

.filestatus {
    width: 100px;
    text-align: center;
}

.fileprogress {
    text-align: center;
}

.domprogress {
    width: 320px;
}

.domsize {
    display: block;
}

#tdmsg {
    text-align: center;
}

#fileselect {
    display: none;
}

span.domtime {
    display:block;
}
```


### HTML 

在 **temples** 中创建上传文件界面  **upload.html**


```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <link rel="stylesheet" href="{ { static_url('css/stylesheet.css') } }">
</head>
<body>
    <div class="container">
        <input type="file" name="fileselect" id="fileselect" value="" multiple/>
        <input type="button" id="btnselect" value="选择上传的文件" />
        <input type="button" id="btnupload" value="开始上传" />
    </div>
    <table cellspacing="0" cellpadding="0" id="filelist">
        <tr>
            <td class="filename">文件名</td>
            <td class="fileprogress">进度</td>
            <td class="filestatus">状态</td>
        </tr>
        <!--<tr><td>人民的名义.avi </td><td><progress value="10" max="100" class="domprogress"></progress><span class="dompercent">10%</span><span class="domsize">0/1.86GB</span></td><td class="filestatus"><span class="domstatus">排队中</span></td></tr>-->
        <tr id="trmsg">
            <td colspan="3" id="tdmsg">请选择要上传的文件! 技术支持QQ：1205352402</td>
        </tr>

    </table>
</body>
<script type="text/javascript" src="{ { static_url('js/jquery-2.0.3.min.js') } }"></script>
<script type="text/javascript" src="{ { static_url('js/spark-md5.js') } }"></script>
</html>
```

这里用到了html5的一个标签 `<progress>`，用来刻画上传的进度


### js

引入jquery库和一些需要的库，这些都放在 **static/js** 目录下

这里需要一个md5加密的js库，点击[这里](https://raw.githubusercontent.com/satazor/SparkMD5/master/spark-md5.js)

下面的代码暂时写在网页中，用来控制界面样式，当选中上传的文件时，界面会有一些变化。


```html
<script type="text/javascript">
    $("#btnselect").click(function() {
        $("#fileselect").click();
    });
    $("#fileselect").change(function() {
        var files = this.files;
        if(files.length > 0) {
            $("#trmsg").remove();
            $(files).each(function(index, item) {
                console.log(index, item);
                var filesize = 0;
                if((item.size / 1024 / 1024 / 1024) >= 1) {
                    filesize = (item.size / 1024 / 1024 / 1024).toFixed(2) + "GB"; // b=>kb=>mb=>gb
                } else if((item.size / 1024 / 1024 / 1024) < 1 && (item.size / 1024 / 1024) >= 1) {
                    filesize = (item.size / 1024 / 1024).toFixed(2) + "MB";
                } else if((item.size / 1024 / 1024) < 1 && (item.size / 1024) >= 1) {
                    filesize = (item.size / 1024).toFixed(2) + "KB";
                } else {
                    filesize = item.size + "B";
                }

                var htmlstr = '<tr><td>' + item.name + '</td><td><progress value="0" max="100" class="domprogress"></progress><span class="dompercent"> 0/'+filesize+'</span><span class="domtime">总共耗时：0 秒</span></td><td class="filestatus"><span class="domstatus">排队中</span></td></tr>';
                $("#filelist").append(htmlstr);

            });
        }
    });
</script>
```


上完css和js，选中需要上传的文件之后的界面是这样的

![Image 2.png](https://i.loli.net/2017/07/28/597aff863150c.png)

当然也可以选择多个文件。


## 上传文件初次尝试

### 前台js代码，实现异步提交文件()在上面js代码后面继续写

#### **开始上传** 

按钮点击事件


```js
$("#btnupload").click(function () {
    var files = $("#fileselect")[0].files;
    $(files).each(function (index) {
        yyupload(files[index]);
    });
});
```

获取所有需要上传的文件，对每一个文件进行处理，将文件传给 `yyupload()`函数进行处理。

```js
function yyupload(file) {
    upload(file);
}
```


这里应给多文件进行一些处理，现在为了实现简单的文件上传，所以 `yyupload()`对文件暂时无处理，直接传给上传函数 `upload()`


```js
function upload(file) {
    var k = 0;
    var fd = new FormData();
    fd.append("file", file);
    fd.append("filename", file.name);
    fd.append("filesize", file.size);

    var xhr = new XMLHttpRequest();
    xhr.open("post", "/uploadfile", true);
    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4 && xhr.status === 200){
            console.log("上传成功");
            k = 0;
        }else if (xhr.status === 500){
            setTimeout(function () {
                if(k < 3){
                    console.log("sendfinish");
                }
                k++;
            }, 3000);
        }
    };
    xhr.send(fd);
}
```

`upload()`函数中，使用了 `FormData`对象和 `XMLHttpRequest`对象。([参考这里](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/Using_FormData_Objects))

- *FormData* 通过FormData对象可以组装一组用 XMLHttpRequest发送请求的键/值对。它可以更灵活方便的发送表单数据，因为可以独立于表单使用
- *XMLHttpRequest* 是一个API, 它为客户端提供了在客户端和服务器之间传输数据的功能。它提供了一个通过 URL 来获取数据的简单方式，并且不会使整个页面刷新。这使得网页只更新一部分页面而不会打扰到用户。XMLHttpRequest 在 AJAX 中被大量使用。

### 后台处理

#### 处理视图 `UploadJobHandler`

在 **views.py**中创建处理视图 `UploadJobHandler`


```python

from settings import BASE_DIR

class UploadJobHandler(tornado.web.RequestHandler):

    def post(self):
        file_metas = self.request.files["file"]
        if len(file_metas) <= 0:
            self.write("获取服务器上传文件失败！")

        metas = file_metas[0]
        filename = self.get_argument("filename")
        tempfilename = filename + ".part"
        newname = os.path.join(BASE_DIR, "file_upload/static/upload/" + tempfilename)
        with open(newname, "wb+") as f:
            f.write(metas["body"])
        self.write("finished!")
```


这里将上传的文件添加后缀 **.part**，放在 `static/upload/` 下。

#### 添加路由 `/uploadfile`

在 **urls.py** 添加 `/uploadfile` 路由，其对应的视图为 `UploadJobHandler`。添加之后 **urls.py**中代码如下：


```python
# coding: utf-8

import tornado.web

from settings import settings
from views import MainHandler, UploadJobHandler

application = tornado.web.Application([
    (r'^/', MainHandler),
    (r'^/uploadfile$', UploadJobHandler),

], **settings)
```

代码编写完成之后，重启服务。此时文件结构如下图：
![Image 3.png](https://i.loli.net/2017/07/28/597b0ade0d554.png)