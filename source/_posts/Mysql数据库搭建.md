---
title: Mysql数据库搭建
date: 2024-04-07
tags: [数据库, mysql]
---


下载介质：https://downloads.mysql.com/archives/community/



1. 解压Mysql压缩包到/usr/local/mysql下：

```
tar -zxvf mysql_5.7.44_linux_x86_64.tar.gz -C /usr/local/
cd /usr/local/
mv mysql-5.7.44-linux-glibc2.12-x86_64 mysql
```

2. 创建mysql用户组及其用户

useradd -g表示把mysql用户添加到mysql用户组中



```
groupadd mysql
useradd -g mysql mysql
```

3. 创建data目录和tmp目录

```
mkdir /usr/local/mysql/data
mkdir /usr/local/mysql/tmp
```

4. 初始化MySQL配置表

```
cd /usr/local/mysql
bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data  
```

初始化之后，会在最后一行看到root账号的初始密码。

有可能报`bin/mysqld: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory`错误。需要安装libaio

```
# centos
yum install -y libaio numactl

# debian
apt install libaio-dev
```

5. 创建配置文件/etc/my.cnf

创建配置文件，并写入以下内容：

```
[client]
socket = /usr/local/mysql/tmp/mysql.sock

[mysqld]
basedir = /usr/local/mysql
datadir = /usr/local/mysql/data
port = 3306
socket = /usr/local/mysql/tmp/mysql.sock

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
```

6. 更改文件权限

```
chown -R mysql /usr/local/mysql/data
chown -R mysql /usr/local/mysql/tmp
```

7. 启动Mysql服务（这步可以不做）

```
./bin/mysqld --defaults-file=/etc/my.cnf --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
```

输出，则表示Mysql服务配置成功：



```
2024-04-03T08:53:44.133293Z 0 [Note] Skipping generation of RSA key pair as key files are present in data directory.
2024-04-03T08:53:44.134295Z 0 [Note] Server hostname (bind-address): '*'; port: 3306
2024-04-03T08:53:44.134347Z 0 [Note] IPv6 is available.
2024-04-03T08:53:44.134355Z 0 [Note]   - '::' resolves to '::';
2024-04-03T08:53:44.134380Z 0 [Note] Server socket created on IP: '::'.
```

8. 建立软连接

```
ln -s /usr/local/mysql/bin/mysql  /usr/local/bin
ln -s /usr/local/mysql/bin/mysqladmin  /usr/local/bin
ln -s /usr/local/mysql/bin/mysqld_safe  /usr/local/bin
```

9. mysqld服务加入开机自启动项

把mysql下的support-files/mysql.server服务脚本放到系统服务，并设置运行权限

```
cp support-files/mysql.server /etc/init.d/mysqld
chmod +x /etc/init.d/mysqld
/etc/init.d/mysqld start
```

这样就可以使用`service mysqld start`启动服务。

10. 修改Mysql密码

使用mysql连接数据库，密码为初始化MySQL配置时，输出的root初始密码：

```
mysql -u root -p
mysql> alter user root@localhost identified by 'mysql';
```

说明：如果使用mysql命令报错：`mysql: error while loading shared libraries: libncurses.so.5: cannot open shared object file: No such file or directory` 则需要安装相关依赖：



```bash
# centos

yum install -y ncurses-compat-libs

# debian
apt install libncurses5
```



补充：

如果在初始化 MySQL 数据库后忘记了 root 用户的密码，通过以下步骤重置：



1. 停止正在运行的 MySQL 服务: `service mysqld stop`
2. 安全模式启动 MySQL 服务: `./bin/mysqld_safe --skip-grant-tables --skip-networking &`
3. 不需要密码就能以 root 用户身份登录到 MySQL 了：`mysql -u root `
4. 不需要密码就能以 root 用户身份登录到 MySQL 了：

```
FLUSH PRIVILEGES;
ALTER USER 'root'@'localhost' IDENTIFIED BY 'mysql';
```

5. 需要退出 MySQL 命令行，然后停止安全模式下的 MySQL 服务，再以正常模式启动 MySQL 服务:

```
# 需要输入新设置的密码，这里示例设置的是mysql
./bin/mysqladmin -u root -p shutdown

# 启动mysql服务
service mysqld start
```
