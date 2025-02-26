---

title: jmpy对Python项目代码加固
date: 2025-02-25
tags: [ 工具 ]

---

- 当前使用Python 版本 3.11.2
- 项目地址：[https://github.com/Boris-code/jmpy](https://github.com/Boris-code/jmpy)

安装 jmpy 包：

```bash
pip install jmpy3
```

执行：

```bash
jmpy -i xxx.py -o /tmp/

# 使用加固后的so文件替换源py文件，代码正常运行：
mv /tmp/dist/xxx.so ./
rm -rf xxx.py
```

如果报错：

```
collection.c:208:27: fatal error: longintrepr.h: No such file or directory
   #include "longintrepr.h"
                           ^
compilation terminated.
error: command '/usr/bin/gcc' failed with exit code 1
```

解决方案：`pip install Cython==0.29.32`

提示：Python 会对 `.py` 文件生成 `.pyc ` 文件，所以加密要在运行前进行，或者加密完成后再删除 `__pycache__/xxx.pyc ` 文件