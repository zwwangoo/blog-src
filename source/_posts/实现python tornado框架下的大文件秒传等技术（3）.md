---
title: 实现python tornado框架下的大文件秒传等技术（3）
date: 2017-07-29
tags: [tornado, python, 技术]
---

这一步实现了文件上传(包括批量)、断点续传、大文件(伪)秒传。但仍然存在不足，待改进。


整体实现的思路是这样：

- 首先是获取文件加密字符串
- 根据md5检查文件的状态：已存在、未完成、不存在。
- 对于已存在的文件，这是直接就显示上传完成。
- 未完成文件和不存在的文件进行分片上传。
- 后台进行文件的保存方式是追加的方式，生成临时文件.part。
- 上传完成之后生成md5后缀为.ok的空文件，待检查使用。同时经上传的临时文件改成原来的名字。

<!--more-->

## 获取文件加密字符串

更改`yyupload()`的参数个数，一下所有的js函数，均写在该函数内部(注意缩进)


```js
function yyupload(file, dommsg, dompercentmb, domprogress, domtime, fn) {
    var startTime = new Date();
    calculate(file);
    // 获取文件加密字符串
    function calculate(file) {

        var fileReader = new FileReader();
        var chunkSize = 1024 * 1024 * 7;  // 每次读取7M
        var chunkCount = Math.ceil(file.size / chunkSize);  // 向上取整
        var currentChunk = 0;  // 当前索引块
        var spart = new SparkMD5();

        loadNext();

        fileReader.onload = function (e) {
            dommsg.text("正在检查文件：" + (currentChunk + 1) + "/" + chunkCount);
            spart.appendBinary(e.target.result);
            currentChunk++;
            if(currentChunk < chunkCount){
                loadNext();
            }else {
                var md5value = spart.end();
                console.log("文件加密结束，密钥为：" + md5value);
                checkfile(md5value, file)
            }
        };

        function loadNext() {
            var start = currentChunk * chunkSize;
            var end = start + chunkSize >= file.size ? file.size : start + chunkSize;
            fileReader.readAsBinaryString(file.slice(start, end))
        }
    }

}
```

文件加密结束后，根据MD5密钥，调用`checkfile()`函数检查服务器上有没有该文件

## 检查服务器上是否存在文件

这个检查函数是比较重要的分叉口，根据检查的结果，对文件的处理是不同的。


```js
    function checkfile(md5value, file){
        var fd = new FormData();
        fd.append("filename", file.name);
        fd.append("md5value", md5value);
        var xhr = new XMLHttpRequest();
        xhr.open("post", "/checkfile", true);
        
        xhr.onreadystatechange = function (res) {
            if(xhr.readyState === 4 && xhr.status === 200){
                var jsonbj = JSON.parse(xhr.responseText);

                switch (jsonbj.flag){
                    case 0:
                    case 1: doUpload(file, md5value, jsonbj.startindex); break;
                    case 2: secondUpload(file); break;
                }
                repeatcount = 0;
            } else if(xhr.status === 500) {
                setTimeout(function() {
                    if(repeatcount < 3) {
                        checkfile(md5value, file);
                    }
                    repeatcount++;
                }, 3000);
            }
        };
        xhr.send(fd);
    }
```


`swich()`处理传回来的参数：

- 0 服务器上不存在该文件
- 1 服务器上存在该文件，但是并未上传完成
- 2 服务器上存在完整的文件，不需要再上传了

对于不需要上传的文件，我们可以之后显示上传完成，造成“秒传”的景象。


```js
    //实现秒传功能
    function secondUpload(file)
    {
        var timerange = (new Date().getTime() - startTime.getTime()) / 1000;
        domtime.text("耗时" + timerange + "秒");
        //显示结果进度
        var percent =100;
        dommsg.text(percent.toFixed(2) + "%");
        domprogress.val(percent);
        var total = file.size;
        if (total > 1024 * 1024 * 1024) {
            dompercentmb.text((total / 1024 / 1024 / 1024).toFixed(2) + "GB/" + (total / 1024 / 1024 / 1024).toFixed(2) + "GB");
        } else if (total > 1024 * 1024) {
            dompercentmb.text((total / 1024 / 1024).toFixed(2) + "MB/" + (total / 1024 / 1024).toFixed(2) + "MB");
        } else if (total > 1024 && total < 1024 * 1024) {
            dompercentmb.text((total / 1024).toFixed(2) + "KB/" + (total / 1024).toFixed(2) + "KB");
        } else {
            dompercentmb.text((total).toFixed(2) + "B/" + (total).toFixed(2) + "B");
        }

    }
```


编写**views.py**文件中的`CheckFileHandler`


```python
class CheckFileHandler(tornado.web.RequestHandler):

    def post(self):
        md5value = self.get_argument("md5value")
        filename = self.get_argument("filename")
        path_part = os.path.join(BASE_DIR, "file_upload/static/upload/" + md5value + ".part")
        path_ok = os.path.join(BASE_DIR, "file_upload/static/upload/" + md5value + ".ok")

        if os.path.isfile(path_ok):  # 文件上传结束
            flag = 2
            ret = {"flag": flag}
        elif os.path.isfile(path_part):
            flag = 1
            startindex = os.path.getsize(path_part)
            ret = {"flag": flag, "startindex": startindex}
        else:
            flag = 0
            ret = {"flag": flag, "startindex": 0}

        self.write(ret)
```


编写**urls.py**文件中的路由`/checkfile`


```python
    (r'^/checkfile$', CheckFileHandler),
```

当然别忘了`from views import CheckFileHandler,`


## 上传文件处理

这一块是处理的核心，js将文件分片，传给后台进行保存。不说废话，上代码：


```js
    function doUpload(file, md5value, startindex) {
        var reader = new FileReader();
        var step = 1024 * 400;  // 每次读取400KB
        var cuLoaded = startindex;
        var total = file.size;

        reader.onload = function (e) {
            var result = reader.result;
            var loaded = e.loaded;
            uploadFile(result, cuLoaded, function () {
                cuLoaded += loaded;
                var timerange = (new Date().getTime() - startTime.getTime()) / 1000;

                if (total > 1024 * 1024 * 1024) {
                    dompercentmb.text((cuLoaded / 1024 / 1024 / 1024).toFixed(2) + "GB/" + (total / 1024 / 1024 / 1024).toFixed(2) + "GB");
                } else if (total > 1024 * 1024) {
                    dompercentmb.text((cuLoaded / 1024 / 1024).toFixed(2) + "MB/" + (total / 1024 / 1024).toFixed(2) + "MB");
                } else if (total > 1024 && total < 1024 * 1024) {
                    dompercentmb.text((cuLoaded / 1024).toFixed(2) + "KB/" + (total / 1024).toFixed(2) + "KB");
                } else {
                    dompercentmb.text((cuLoaded).toFixed(2) + "B/" + (total).toFixed(2) + "B");
                }
                domtime.text("耗时" + timerange + "秒");
                domtime.text("耗时" + timerange + "秒");
                if (cuLoaded < total) {
                    readBlob(cuLoaded);
                } else {
                    console.log('总共用时：' + timerange);
                    cuLoaded = total;
                    sendfinish(); //告知服务器上传完毕
                    domtime.text("上传完成,总共耗时" + timerange + "秒");
                }
                //显示结果进度
                var percent = (cuLoaded / total) * 100;
                dommsg.text(percent.toFixed(2) + "%");
                domprogress.val(percent);
            });
        };
        readBlob(cuLoaded);
        //指定开始位置，分块读取文件
        function readBlob(start) {
            //指定开始位置和结束位置读取文件
            var end = start + step >= file.size ? file.size : start + step;
            var blob = file.slice(start, end); //读取开始位置和结束位置的文件
            reader.readAsArrayBuffer(blob); //读取切割好的文件块
        }
        //继续
        function containue() {
            readBlob(cuLoaded);
        }

        ……
    }
```


文件上传结束时，通知服务器：（**……**表示连接着的）


```js
        ……

        var k = 0;
        function sendfinish() {
            var fd = new FormData();
            fd.append("filename", file.name);
            fd.append("md5value", md5value);
            fd.append("totalsize", file.size);

            var xhr = new XMLHttpRequest();
            xhr.open("post", "/finishupload", true);
            xhr.onreadystatechange = function () {
                if (xhr.readyState === 4 && xhr.status === 200){
                    if(fn){
                        fn();
                    }
                    k = 0;
                }else if (xhr.status === 500){
                    setTimeout(function () {
                        if(k<3){
                            sendfinish();
                        }
                        k++;
                    }, 300);
                }
            };
            xhr.send(fd);
        }
        ……
```

将分片文件上传到服务器：


```js
        ……
        var m = 0;
        function uploadFile(result, startIndex, onSuccess) {
            var blob = new Blob([result]);
            var fd = new FormData();
            fd.append("file", blob);
            fd.append("md5value", md5value);
            fd.append("filename", file.name);
            fd.append("filesize", file.size);
            fd.append("loaded", startIndex);

            var xhr = new XMLHttpRequest();
            xhr.open("post", "/uploadfile", true);
            xhr.onreadystatechange = function () {
                if (xhr.readyState === 4 && xhr.status === 200){
                    m = 0;
                    if(onSuccess) onSuccess();
                }else if (xhr.status === 500){
                    setTimeout(function () {
                        if(m < 3){
                            console.log("sendfinish");
                        }
                        m++;
                    }, 1000);
                }
            };
            xhr.send(fd);
        }
```


这里有两个后台处理程序。

在views.py中编写 `UploadJobHandler`、 `FinishUpload`


```python
class UploadJobHandler(tornado.web.RequestHandler):

    def post(self):
        file_metas = self.request.files["file"]
        if len(file_metas) <= 0:
            self.write("获取服务器上传文件失败！")

        metas = file_metas[0]
        md5value = self.get_argument("md5value")
        tempfilename = md5value + ".part"
        newname = os.path.join(BASE_DIR, "file_upload/static/upload/" + tempfilename)

        with open(newname, "ab") as f:  # 以二机制方式追加
            f.write(metas["body"])
        self.write("finished!")

    def get(self, *args, **kwargs):
        self.write("ok")


class FinishUpload(tornado.web.RequestHandler):

    def post(self, *args, **kwargs):
        md5value = self.get_argument("md5value")
        filename = self.get_argument("filename")
        totalsize = self.get_argument("totalsize")

        path_part = os.path.join(BASE_DIR, "file_upload/static/upload/" + md5value + ".part")
        path_ok = os.path.join(BASE_DIR, "file_upload/static/upload/" + md5value + ".ok")
        old_name = os.path.join(BASE_DIR, "file_upload/static/upload/" + filename)

        with open(path_ok, "w") as f:
            print "创建ok文件"
        os.rename(path_part, old_name)
        self.write("{'data': 'ok'}")
```

注意：

- 文件在上传时，临时文件名为 md5value + ".part"
- 上传成功后，创建空文件 md5value + ".ok"
- 文件保存的时候，这里是使用二进制方式追加，否则保存的文件不可用了。


编写**urls.py**文件中的路由`/uploadfile`、`/finishupload`

```python

    from views import UploadJobHandler, FinishUpload

    (r'^/uploadfile$', UploadJobHandler),
    (r'^/finishupload$', FinishUpload),
```


效果如下图


![1.png](https://i.loli.net/2017/07/30/597d5287d6435.png)
![Image 2.png](https://ooo.0o0.ooo/2017/07/30/597d5287e975e.png)
![Image 3.png](https://i.loli.net/2017/07/30/597d5287ea9ce.png)


## 不足

- 大文件上传，速度仍是不理想
- 对于大文件，断点续传，可能导致文件不能用
- 上传速度受网速影响

接下来该改进了，到此，以上参考的是[大文件分块上传技术分享(c#实现)](https://www.idaima.com/article/17099)，特此感谢分享。