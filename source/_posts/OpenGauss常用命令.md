---

title: OpenGauss常用命令
date: 2024-08-20
tags: [ 数据库 ]

---

使用odbc驱动：

配置 `odbcinst.ini`

```
[GaussMPP]
Description = OpenGauss MPP
Driver=/opt/opengauss-odbc/odbc/lib/psqlodbcw.so
Setup=/opt/opengauss-odbc/odbc/lib/psqlodbcw.so
```

使用 `isql` 验证连接

```
isql -v -k 'DRIVER={GaussMPP};SERVER=172.16.52.28;PORT=15400;DATABASE=postgres;UID=drcc;PWD=Hzmc321#'
```

使用 `pyodbc` 验证连接

```

import pyodbc

# 创建连接
conn_str = 'DRIVER={GaussMPP};SERVER=192.168.236.28;PORT=15400;DATABASE=postgres;UID=drcc;PWD=Hzmc321#'

conn = pyodbc.connect(conn_str)
cursor = conn.cursor()
cursor.execute('select version()')
print(cursor.fetchall())
cursor.close()
conn.close()
```

切换到 omm 用户：

```
su - omm
```

重启服务：

```
gs_om -t restart
```

客户端使用：

```
gsql -d postgres -U drcc
```

账号解锁：

```
gsql -d postgres
openGauss=# alter user drcc account unlock;
openGauss=# \q
```

修改账号密码：

```
openGauss=# alter user drcc password 'Hzmc321#';
```

修改加密方式：

```
gsql -d postgres
openGauss=# show password_encryption_type;
 password_encryption_type
--------------------------
 2
(1 row)
```

```
gs_guc reload -N all -I all -c "password_encryption_type=2"
```

> 当参数 password_encryption_type 设置为 0 时，表示采用 md5 方式对密码加密。Md5 为不安全的加密算法，不建议使用。
> 当参数 password_encryption_type 设置为 1 时，表示采用 sha256 和 md5 方式对密码加密。其中包含 md 5 为不安全的加密算法，不建议使用。
> 当参数 password_encryption_type 设置为 2 时，表示采用  sha256 方式对密码加密，为默认