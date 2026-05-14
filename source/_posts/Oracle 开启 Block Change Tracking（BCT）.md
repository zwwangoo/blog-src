---
title: Oracle 开启 Block Change Tracking（BCT）
date: 2026-05-14
tags:
  - 数据库
---
## 备份前开启 BCT 的好处

在 Oracle 数据库中，开启 Block Change Tracking（BCT）可以显著提升 RMAN 增量备份的效率。BCT 会记录自上次备份以来发生变化的数据块，RMAN 在执行增量备份时无需再扫描整个数据文件，只需要读取发生变化的块，从而减少备份扫描时间、降低磁盘 I/O 压力，并缩短备份窗口。

对于数据量较大、增量备份频繁的生产库，建议在执行增量备份前提前开启 BCT，以提升备份性能和稳定性。需要注意的是，BCT 主要对 RMAN 增量备份有明显优化效果，对全量备份帮助不大。

开启后可通过 `V$BLOCK_CHANGE_TRACKING` 视图确认状态：

```sql
SELECT status, filename
FROM v$block_change_tracking;
```

如果状态为 `ENABLED`，说明 BCT 已开启；如果为 `DISABLED`，则需要根据实际存储路径重新配置并开启。

### 1. 登录数据库主机并切换 Oracle 用户

```bash
su - oracle
```

确认 Oracle 环境变量：

```bash
echo $ORACLE_SID
echo $ORACLE_HOME
```

如果未设置，需要根据实际实例进行设置，例如：

```bash
export ORACLE_SID=orcl
export ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
```

---

### 2. 进入 SQL*Plus

```bash
sqlplus / as sysdba
```

---

### 3. 查看当前 BCT 状态

```sql
SELECT status, filename FROM v$block_change_tracking;
```

如果看到如下结果，说明 BCT 未开启：

```text
STATUS
----------
DISABLED
```

---

### 4. 开启 Block Change Tracking

#### 情况一：数据库已配置 `db_create_file_dest`

先查看参数：

```sql
SHOW PARAMETER db_create_file_dest;
```

如果该参数有值，例如 `+DATA` 或某个数据文件目录，可以直接执行：

```sql
ALTER DATABASE ENABLE BLOCK CHANGE TRACKING;
```

#### 情况二：未配置 `db_create_file_dest`

如果没有配置 `db_create_file_dest`，需要手动指定 BCT 文件路径。

普通文件系统示例：

```sql
ALTER DATABASE ENABLE BLOCK CHANGE TRACKING
USING FILE '/u01/app/oracle/oradata/ORCL/block_change_tracking.f';
```

ASM 示例：

```sql
ALTER DATABASE ENABLE BLOCK CHANGE TRACKING
USING FILE '+DATA/ORCL/block_change_tracking.f';
```

#### 注意事项

- 指定路径需要 Oracle 用户具备写权限。
- RAC 环境建议将 BCT 文件放在共享存储或 ASM 上。
- 不建议放在临时目录、备份挂载目录或可能被清理的目录中。
- BCT 文件通常很小，但必须长期存在。

---

### 5. 再次验证 BCT 状态

```sql
SELECT status, filename FROM v$block_change_tracking;
```

期望结果示例：

```text
STATUS    FILENAME
--------- ----------------------------------------
ENABLED   /u01/app/oracle/oradata/ORCL/block_change_tracking.f
```

或者显示 ASM 路径。

当前系统新增的校验要求如下查询必须返回 `ENABLED`：

```sql
SELECT status FROM v$block_change_tracking;
```

期望结果：

```text
ENABLED
```

---

### 6. 关闭 BCT，仅供参考

如需关闭 Block Change Tracking，可以执行：

```sql
ALTER DATABASE DISABLE BLOCK CHANGE TRACKING;
```
