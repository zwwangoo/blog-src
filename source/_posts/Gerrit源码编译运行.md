---
Title: Gerrit 源码编译并运行
Date: 2024-12-18
Tags: [ 技术文档 ]
---

0. Docker 镜像拉取运行并换源：
```bash
docker run -it -p 8080:8080 ubuntu:20.10 bash

# 换国内源
mv /etc/apt/sources.list /etc/apt/sources.list.bak
echo "deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse" | tee /etc/apt/sources.list
```

1. 安装依赖：
```
apt update
apt install -y git openjdk-17-jdk python3 zip curl wget aptitude
```

2. 安装 Bazel
```bash
# 这里使用了 aptitude 进行安装，会检测依赖是否安装，注意看提示的信息进行选择。
aptitude install gnupg
curl -fsSL https://bazel.build/bazel-release.pub.gpg | gpg --dearmor > bazel-archive-keyring.gpg
mv bazel-archive-keyring.gpg /usr/share/keyrings

echo "deb [arch=amd64 signed-by=/usr/share/keyrings/bazel-archive-keyring.gpg] https://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list
apt update
aptitude install bazel=7.2.1

# 安装完成后查看版本
bazel version
```

3. 克隆 Gerrit 代码并构建
```
git clone --recursive https://gerrit.googlesource.com/gerrit
cd gerrit
bazel build :release
```

4. 初始化 Gerrit Site 并启动
```
java -jar bazel-bin/release.war init -d </path/to/gerrit/site>

# 可能初始化后就启动了，下面命令可以执行下：
</path/to/gerrit/site>/bin/gerrit.sh start
```
5. 访问 Web UI `http://localhost:8080`