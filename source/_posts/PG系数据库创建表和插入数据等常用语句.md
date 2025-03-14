---

title: PG系数据库创建表和插入数据等常用语句
date: 2024-12-27
tags: [ 数据库 ]

---

## 创建数据库表

在 gsql 提示符下，你可以使用 CREATE TABLE 语句来创建表。以下是一个示例，创建一个名为 employees 的表：

```
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    position VARCHAR(100),
    salary NUMERIC(10, 2)
);

```

## PG 开启归档:

编辑 `$PGDATA/postgresql.conf` 设置以下参数：

```
# 开启归档模式
archive_mode = on
   
# 指定归档命令，用于将WAL文件复制到归档目录
archive_command = 'cp %p /path/to/archive/%f'
```

- `/path/to/archive/` 应替换为实际的归档目录路径

重启数据库:

```
pg_ctl restart
```

查看归档模式：

```
$ psql -p15432 -Urepmgr

repmgr=# show wal_log_hints;
 wal_log_hints
---------------
 on
(1 row)

repmgr=# show archive_mode;
 archive_mode
--------------
 on
(1 row)

repmgr=# show archive_command;
         archive_command
---------------------------------
 cp %p /u01/pg14/data/pg_achr/%f
(1 row)

```