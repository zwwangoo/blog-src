---
title: Ubuntu编译安装Aria2 – 突破百度网盘下载限速
date: 2018-11-30
tags: [Ubuntu, 服务器]
---

# 下载源码

安装一些必要的依赖环境

```
apt install -y libcurl4-openssl-dev libevent-dev \
                ca-certificates libssl-dev pkg-config \
                build-essential intltool libgcrypt-dev \
                libssl-dev libxml2-dev
```

下载最新的aria2的源码：

```
wget --no-check-certificate https://github.com/aria2/aria2/releases/download/release-1.31.0/aria2-1.31.0.tar.gz
tar zxf aria2-1.31.0.tar.gz
cd ./aria2-1.31.0
```

# 修改源码

Aria2的参数中`-max-server-connection`和`-min-split-files`很大程度上决定了网盘的下载速度。
在这里我将`-max-server-connection`最高设置为256，`-min-split-files`最小设置为256k。

<!--more-->

```
    #aira2的基本配置选项大多都存储在文件OptionHandlerFactory.cc中
    vi src/OptionHandlerFactory.cc

    #定位到441行
    #将服务器最大连接数16修改为256
    OptionHandler* op(new NumberOptionHandler(PREF_MAX_CONNECTION_PER_SERVER,
                                              TEXT_MAX_CONNECTION_PER_SERVER,
                                           // "1", 1, 16, 'x'));
                                              "1", 1, 256, 'x'));
    #定位到第503行
    #将最文件分片大小设置为256_k
    #到此源代码需要修改的地方改完了
    OptionHandler* op(new UnitNumberOptionHandler(
    //PREF_MIN_SPLIT_SIZE, TEXT_MIN_SPLIT_SIZE, "1M", 1_m, 1_g, 'k'));
     PREF_MIN_SPLIT_SIZE, TEXT_MIN_SPLIT_SIZE, "1M", 256_k, 1_g, 'k'));
```

# 编译前的参数设置

```
./configure
```

编译

```
make
```

编译完成之后

```
cp src/aria2c /usr/local/bin
```

# 查看安装是否成功

```
aria2c -v
```

如果能显示版本号，就表示成功。

# 配置

在/etc里新建一个aria2的目录,新建配置文件aria2.conf

```
mkdir -p /etc/aria2
vi /etc/aria2/aria2.conf
```

插入以下内容：

```
dir=/home/wen/Downloads  # 这里需要改成绝对路径
disable-ipv6=true
enable-rpc=true
rpc-allow-origin-all=true
rpc-listen-all=true
rpc-listen-port=6800
continue=true
input-file=/etc/aria2/aria2.session
#rpc-user=admin
#rpc-passwd=password
save-session=/etc/aria2/aria2.session
save-session-interval=7200
max-concurrent-downloads=20
max-connection-per-server=256
min-split-size=256k
#log=/var/log/aria2/aria2.log
# Complete delete .aria2 files
on-download-complete=/etc/aria2/delete_aria2
max-overall-upload-limit=5K
max-upload-limit=5K
follow-torrent=true
#BT
bt-request-peer-speed-limit=200K
#PT download
bt-max-peers=48
listen-port=26834
enable-dht=false
bt-enable-lpd=false
enable-peer-exchange=false
user-agent=uTorrent/341(109279400)(30888)
peer-id-prefix=-UT341-
seed-ratio=0
force-save=true
bt-hash-check-seed=true
bt-seed-unverified=true
bt-save-metadata=true
```

新建一个aria2.session，用于存储正在下载的一些信息

```
touch /etc/aria2/aria2.session
```

# 启动

启动文件配置

```
#!/bin/sh
### BEGIN INIT INFO
# Provides:          aria2
# Required-Start:    $remote_fs $network
# Required-Stop:     $remote_fs $network
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Aria2 Downloader
### END INIT INFO
case "$1" in
start)
    echo -n "Starting aria2cn"
    sudo -u root aria2c --conf-path=/etc/aria2/aria2.conf -D
    #sudo -u后面的是你正在使用的用户名，因为我用的root
;;
stop)
    echo -n "Shutting down aria2c "
    killall aria2c
;;
restart)
    echo -n "Shutting down aria2c  "
    killall aria2c
sleep 3
echo -n "Starting aria2c"
    sudo -u root aria2c --conf-path=/etc/aria2/aria2.conf -D
    #同上面的一样，根据自己的用户名改root。
;;
*)
    echo 'Usage:' `basename $0` '[option]'
    echo 'Available option:'
    for option in start stop restart
    do
    echo '  -' $option
    done
;;
esac
```

给启动文件一个权限以及开机自动运行aria2

```
sudo chmod 755 /etc/init.d/aria2c
update-rc.d aria2c defaults
```

启动aria2

```
sudo service aria2c start
```

# 参考文章

- [https://www.lucktang.com/2559.html](https://www.lucktang.com/2559.html)
- [https://www.easegamer.com/?p=483](https://www.easegamer.com/?p=483)
