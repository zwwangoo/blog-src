---
title: 源码安装Postgresql14.4
date: 2022-12-07
tags: [数据库,PG,debian]
---

下载源码包 [https://www.postgresql.org/ftp/source/v14.4/](https://www.postgresql.org/ftp/source/v14.4/)

## 源码安装

### 0、安装依赖

```
shell> apt install gcc g++ make libreadline-dev zlib1g zlib1g-dev libkrb5-dev libssl-dev libpam0g-dev libxml2-dev libxslt-dev libldap2-dev gettext tcl8.6-dev tcl-dev libperl-dev python-dev -y
```

离线安装时，需要使用iso镜像安装依赖(TODO)

### 1、解压 PG14源码

```
shell> mkdir -p /pg/PG14.4
shell> tar -xzvf postgresql-14.4.tar.gz -C /pg/PG14.4
```

### 2、创建用户并配置环境变量

添加安装用户和安装目录权限赋予

```
shell> useradd -m -s /bin/bash postgres
shell> mkdir /pg/PG14.4
shell> chown -R postgres:postgres /pg/PG14.4
```

配置环境变量

```
shell> su - postgres
shell> vi ~/.bash_profile
# 添加以下：
export PATH=$PATH:$HOME/bin:/pg/PG14.4/bin/

shell> source ~/.bash_profile
```

### 3、编译安装 PG

<!--more-->

```
shell> su – postgres
shell> cd /pg/PG14.4/postgresql-14.4/
shell> ./configure --prefix=/pg/PG14.4 \
--enable-nls \
--with-perl --with-python --with-tcl --with-gssapi --with-openssl --with-pam \
--with-ldap --with-libxml --with-libxslt

shell> make world
shell> make install-world
```

### 4、初始化 PG 数据库

主机上部署多个实例时，需要指定不同的数据目录。

```
shell> su – postgres
shell> mkdir -p /pg/PG14.4/data
shell> /pg/PG14.4/bin/initdb -D /pg/PG14.4/data
shell> /pg/PG14.4/bin/pg_ctl -D /pg/PG14.4/data -l logfile start
shell> /pg/PG14.4/bin/createdb test
shell> /pg/PG14.4/bin/psql test #登陆 test 库,\q 退出
```

### 5、重启 PG 实例测试

```
shell> su – postgres 
shell> /pg/PG14.4/bin/pg_ctl stop -D /pg/PG14.4/data
shell> /pg/PG14.4/bin/pg_ctl start -D /pg/PG14.4/data
```

PostgreSQL 数据库默认会创建一个 postgres 的数据库用户作为数据库的管理员，默认
密码为空，我们需要修改为指定的密码，这里设定为 postgres.

```
shell> /pg/PG14.4/bin/psql
postgres=# ALTER USER postgres with password 'postgres';
```

## 连接配置

### 1、配置 pg_hba.conf 白名单

```
shell> su - postgres 
shell> vi /pg/PG14.4/data/pg_hba.conf
#添加以下行
host all all 10.211.0.0/0 md5
#禁止超级用户远程连接
host all postgres 0.0.0.0/0 reject

shell> /pg/PG14.4/bin/pg_ctl reload -D /pg/PG14.4/data
```

### 2、修改 listen_addresses 参数

注意：需要重启生效	

```
shell> su - postgres
shell> cd /pg/PG14.4/data
shell> vi postgresql.conf
#修改以下参数，并保存退出
listen_addresses ＝’*’

#重启数据库服务
shell> /pg/PG14.4/bin/pg_ctl restart -D /pg/PG14.4/data
```

### 3、连接测试

```
shell> psql -h 10.211.55.5 -p 5432 -U postgres test
Password for user postgres: 
psql (14.6 (Debian 14.6-1.pgdg110+1), server 14.4)
Type "help" for help.
test=# \conninfo
You are connected to database "test" as user "postgres" on host "10.211.55.5" at port "5432".
```

