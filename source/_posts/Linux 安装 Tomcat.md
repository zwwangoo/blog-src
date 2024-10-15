---

title: Linux 安装 Tomcat
date: 2024-10-14
tags: [ 技术文档 ]

---
1. 安装 JDK[Linux 安装JDK和Java环境变量配置](Linux%20安装JDK和Java环境变量配置.md)
2. 下载 Tomcat，官网: https://tomcat.apache.org 这里实例安装 Tomcat 10.1.31
3. 安装
```
mkdir /usr/local/tomcat
tar -zxvf apache-tomcat-10.1.31.tar.gz -C /usr/local/tomcat
```
4. 启动和停止
```
cd /usr/local/tomcat/apache-tomcat-10.1.31/bin
# 启动
./startup.sh
# 停止
./shutdown.sh
```

5. Tomcat 服务器根目录下的文件夹:

- bin: 存放 Tomcat 的启动、停止等相关命令
- lib: 存放 Tomcat 运行时所需要的 jar 包
- conf: Tomcat 配置文件目录
- logs: Tomcat 运行日志目录
- webapps: 存放运行在 Tomcat 服务器内的应用程序（JavaWeb 应用部署目录）
- work: 存放应用程序运行时动态生成的 java 代码和动态编译的 class 文件
- temp: 存放 Tomcat 运行时产生的临时文件