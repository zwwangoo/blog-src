---
title: Python实现给excel表格设置密码
date: 2019-04-26
tags: [python, java]
---

入职新公司，同事让帮忙写一个Python脚本，实现每天定时获取生产环境的数据生成excel表格，推送到指定邮箱。想了想，不难实现，就答应下来了。但是等写完了才被通知到，邮件携带的excel表格，要设置指定密码，进行保护，顿时懵逼，貌似Python暂时不能实现吧？然后各种资料源码查看，然后确定了确实不能实现，先通知同事一声，说Python实现不了设置密码的功能，然后就只能放弃了。

但是哥不是那种容易放弃的人！旁边坐的朱同学说，我看看Java能否实现。果不其然，Java才是万能的！(拍一下各位Java看官大佬们的马屁)。

于是，才有了这个文章，以下简单整理一下实现的过程，虽然都是从网上参考过来的，但是这个功能点是能够实现的，再次记录分享一下，以供下次参考。准确的说，本文应该是“实战Python通过调用Java的第三方jar包完成对excel表格的密码设置”。下面开始正文。

## 环境搭建

Python的环境和JDK的环境就不用说了，我这里使用的是Python3.6和JDK8。重点是以下的一个Python库和一个Java的jar包。

<!--more-->

### Python的JPype库

JPype是一个能够让 Python 代码方便地调用 Java 代码的工具，从而克服了 Python 在某些领域（如服务器端编程）中的不足。JPype的实际运行环境仍然是python runtime，只是在运行期间启动了一个嵌入的jvm。

#### 安装

通过pip进行安装：

```shell
pip install jpype
```

Ubuntu系统还可以通过apt进行安装

```shell
sudo apt install python-jpype
```

#### 使用

JPype 提供的 `startJVM() `函数的作用是启动 JAVA 虚拟机，所以在后续的任何 JAVA 代码被调用前，必须先调用此方法启动 JAVA 虚拟机。`get_default_jvm_path`方法可以获取当前系统中JVM的所在路径。

```
import jpype
jpype.startJVM(jpype.get_default_jvm_path(), *args)
```

**第三方包的引用**

很多时候，在 Python 项目中需要调用第三方的 Java 扩展包，这也是 JPype 的一个重要用途。

- 通过在 JVM 启动参数增加：`-Djava.class.path = ext_classpath`，实现在 Python 代码中调用已有的 Java 扩展包。
- 通过在 JVM 启动参数增加： `-Djava.ext.dirs = ext_dirs `, 实现在Python 代码中引入 Java 扩展包的其他依赖包。

**关闭JVM**

当使用完 JVM 后，可以通过` jpype.shutdownJVM()` 来关闭 JVM，该函数没有输入参数。

当 Python 程序退出时，JVM 也会自动关闭。

> 注：jpype本身无法多次启动和关闭JVM，在使用是，可以不关闭JVM，在需要启动的地方时进行JVM是否已经启动的判断:
>
> ```python
> if not jpype.isJVMStarted(): 
>      jpype.startJVM()
> ```

#### 示例

这里演示一个最简单的Java示例：

```python
import jpype
if not jpype.isJVMStarted(): 
     jpype.startJVM(jpype.get_default_jvm_path())
     System = jpype.JClass('java.lang.System')
     System.out.println('hello word!')
```

### Java 的jecell.jar

对Java并不是很熟悉，刚刚简单的过了一便基本的语法，更别提对jar包的理解了，简单从网上搜索大概了解jecell是处理excel的jar，可以生成报表，带图片，动态的，还可以在客户不安装OFFICE的情况下使用。可以实现对excel设置密码和去除密码。

可以在这里下载：[链接: https://pan.baidu.com/s/1XllVYN6CkRpB1R16HS_78Q 提取码: 27xw](https://pan.baidu.com/s/1XllVYN6CkRpB1R16HS_78Q)

下面简单的示例：

```java
import java.io.IOException;
import com.jxcell.CellException;
import com.jxcell.View;

/**
 * excel加密、解密 代码
 */
public class EncryptDecryptUtil {

    /**
     * 读取excel，并进行加密
     * 
     * @param url excel文件路径 例："./aaa.xls"
     * @param pwd 加密密码
     */
    public static void encrypt(String url, String pwd) {
        View m_view = new View();
        try {
            // read excel
            m_view.read(url);
            // set the workbook open password
            m_view.write(url, pwd);
        } catch (CellException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * excel 解密
     */
    public static void decrypt(String url, String pwd) {
        View m_view = new View();
        try {
            // read the encrypted excel file
            m_view.read(url, pwd);
            // write without password protected
            m_view.write(url);
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
    }

    public static void main(String args[]) {
        // 下面1与2 两个方法请分开执行，可以看到效果
        //
        // 1. 把./aaa.xls 添加打开密码123
        EncryptDecryptUtil.encrypt("./aaa.xls", "123");
        // 2. 把./aaa.xls 密码123 去除
        // EncryptDecryptUtil.decrypt("./aaa.xls", "123");

    }
}

```

## 实现

经过上面的简单介绍，开始进入主题，先丢代码为敬：`encrypt_xls.py`

```python
import jpype
import os

project_dir = os.path.dirname(os.path.abspath(__file__))

jvm_path = jpype.get_default_jvm_path()
# 这里根据实际的jxcell.jar路径进行配置,我这里的放的位置是本文件同级目录的lib/jxcell.jar
jxcell_path = os.path.join(project_dir, 'lib/jxcell.jar')


def encrypt(url, passwd):
    jpype.startJVM(jvm_path, '-ea', '-Djava.class.path=' + jxcell_path)
    view = jpype.JClass('com.jxcell.View')
    m_view = view()
    m_view.write(url, passwd)
    return url


if __name__ == '__main__':
    # 在当前目录下创建一个test.xls表格，随便输入一些数据，供测试
    encrypt('./test.xls', '123')
```

当前文件有必要说明一下：

- src/
  - lib/jxcell.jar
  - encrypt_xls.py
  - test.xls

运行文件之后，打开test.xls文件会发现已经加密成功。

![文档已加密](https://i.loli.net/2019/04/27/5cc332e6cdc29.png)


本文参考：

- [使用 Python 的 JPype 模块调用 Jar 包](<https://testerhome.com/topics/12394>)
- [[java代码实现对excel加密、解密](https://www.cnblogs.com/haha12/p/4335076.html)](<https://www.cnblogs.com/haha12/p/4335076.html>)
