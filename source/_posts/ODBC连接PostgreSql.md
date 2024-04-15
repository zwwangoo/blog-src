---
title: ODBC连接PostgreSql
date: 2024-04-15
tags: [ODBC,PostgreSql]
---

安装驱动：

```
yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
yum install postgresql15-odbc
```

TODO 离线包

配置 `/etc/odbcinst.ini`

```
[PostgreSQL]
Description=ODBC for PostgreSQL
Driver=/usr/pgsql-15/lib/psqlodbcw.so
Debug=0
CommLog=1
UsageCount=1
```

测试连接：

```
isql -v -k 'DRIVER={PostgreSQL};SERVER=192.168.2.181;PORT=15432;UID=abcdu;PWD=123456;DATABASE=test;'
+---------------------------------------+
| Connected!                            |
|                                       |
| sql-statement                         |
| help [tablename]                      |
| quit                                  |
|                                       |
+---------------------------------------+
```
