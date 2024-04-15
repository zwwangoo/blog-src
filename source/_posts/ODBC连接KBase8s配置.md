---
title: ODBC连接KBase8s配置
date: 2024-04-15
tags: [ODBC,KBbase8s]
---

1、选择对应的操作系统下载驱动：[https://gbasedbt.com/dl/odbc/](https://gbasedbt.com/dl/odbc/)

2、安装GBase 8s数据库连接工具（CSDK），解压到指定目录/opt下，生成/opt/gbase8s-odbc-driver目录

```
tar -zxf GBase8s_3.0-Linux64-ODBC-Driver.tar.gz -C /opt/
```

3、创建必须的环境变量，并使环境生效

```
export GBASEDBTDIR=/opt/gbase8s-odbc-driver
export CSDK_HOME=/opt/gbase8s-odbc-driver
export PATH=$GBASEDBTDIR/bin:$PATH
export LD_LIBRARY_PATH=$GBASEDBTDIR/lib:$GBASEDBTDIR/lib/cli:$GBASEDBTDIR/lib/esql:$LD_LIBRARY_PATH
```

4、修改ODBCINI配置文件，增加GBase 8s数据库连接信息

```
[GBase ODBC DRIVER]
Driver=/opt/gbase8s-odbc-driver/lib/cli/iclis09b.so
Description=GBase ODBC DRIVER
CursorBehavior=0
CLIENT_LOCALE=zh_CN.utf8
DB_LOCALE=zh_CN.utf8
TRANSLATIONDLL=/opt/gbase8s-odbc-driver/lib/esql/igo4a304.so
```

5、通过isql命令检查ODBC配置

（1）配置/opt/gbase8s-odbc-dirver/etc/sqlhosts

```
gbase01 onsoctcp 192.168.236.192 9088
```

（2）测试连接

```
isql -v -k "DRIVER={GBase ODBC Driver};SERVER=gbase01;UID=gbasedbt;PWD=GBase123;DATABASE=sysmaster;"
```

输出表示连接成功：

```
+---------------------------------------+
| Connected!                            |
|                                       |
| sql-statement                         |
| help [tablename]                      |
| quit                                  |
|                                       |
+---------------------------------------+
SQL>
```
