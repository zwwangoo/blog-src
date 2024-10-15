---
title: Linux 安装JDK和Java环境变量配置
date: 2024-10-14
tags:
  - 技术文档
---
下载 JDK

JDK 官网下载地址: https://www.oracle.com/technetwork/java/javase/downloads/index.html

tar.gz 是普通的压缩包文件，直接解压即可。

1、安装 JDK

```

# 通常 JDK 安装到 /usr 目录下
mkdir /usr/java

# 把安装包复制到安装目录下
tar -zxvf jdk-23_linux-x64_bin.tar.gz -C /usr/java

# 进入 java 安装目录
cd /usr/java/jdk-23

```

2、配置环境变量
```
# 打开全局环境变量配置文件
vim /etc/profile

# 添加以下内容：
export JAVA_HOME=/usr/java/jdk-23
export PATH=$JAVA_HOME/bin:$PATH

# 重新加载终端 
source /etc/profile
```

3、检查结果
```
java -version
```