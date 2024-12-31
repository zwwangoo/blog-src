---

title: Oracle设置密码有效期
date: 2024-07-12
tags: [ 数据库 ]

---

1、查看用户的proifle是哪个，一般是default：

```sql
SELECT username,PROFILE,account_status FROM dba_users where username='DRCC';
```

2、查看指定概要文件（如default）的密码有效期设置：

```sql
SELECT * FROM dba_profiles s WHERE s.profile='DEFAULT' AND resource_name='PASSWORD_LIFE_TIME';
```

3、将密码有效期由默认的180天修改成“无限制”：

```sql
ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME UNLIMITED;
```

修改之后不需要重启动数据库，会立即生效。

4、将密码有效期设置为指定天数，如60天:

```sql
alter profile default limit PASSWORD_LIFE_TIME 60;
```

修改之后不需要重启动数据库，会立即生效。

5、修改后，还没有被提示ORA-28002警告的帐户不会再碰到同样的提示；

已经被提示的帐户必须再改一次密码，举例如下：

```shell
sqlplus / as sysdba
```

修改密码：

```sql
alter user drcc identified by <原来的密码> ----不用换新密码
```

例如：

```sql
alter user drcc identified by "okp@admin!123!"
```

https://www.cnblogs.com/songjinduo/p/4642026.html