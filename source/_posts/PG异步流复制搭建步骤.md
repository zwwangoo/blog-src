---
title: PG异步流复制搭建步骤
date: 2022-12-05
tags: [数据库, PG]

---

## 主库操作部分

### 1、创建复制用户

```
postgres=# create user repl replication password 'Abcd321#';
CREATE ROLE
```

### 2、修改主库 pg_hba.conf 白名单文件

将用于流复制数据同步的 repl 用户，添加至 /etc/postgresql/14/main/pg_hba.conf 文件中

```
shell> vi /etc/postgresql/14/main/pg_hba.conf
host replication repl 10.211.55.4/32 md5
```

repl 用户添加之后，及时手动进行生效

```
shell> pg_ctl reload
server signaled
```

### 3、修改参数文件

<!--more-->

修改主库参数文件，添加异步流复制配置相关参数，如下：

```
vi /etc/postgresql/14/main/postgresql.conf
# 这个一般安装服务器之后都会进行配置，检查下。设置为*就运行所有 ip，因为前面 pg_hba.conf 文件中做了对于 IP 的限制，所以这里就设置为*
listen_addresses = '*'
# pg_wal 目录中的过去日志文件段的最小大小，备用服务器需要读取它们进行流式传输复制
wal_keep_size = '1GB'
# 这个设置了可以最多有几个流复制连接，默认 10
max_wal_senders = 32
# 如果有错误的数据复制，是否向主进行反馈
hot_standby_feedback = on
```

异步流复制参数设置完成后，需手动重启实例

```
shell> pg_ctl restart
```

注意：其中只有 listen_address 参数设置需要重启生效，对于其他参数则可以选择 pg_ctl reload 或者 select pg_reload_conf();方式动态生效即可

### 4、从库测试连接

TODO

## 备库操作

### 1、停止备库 PG 服务并清空 data 目录

检查 PG 数据目录，并手动关闭 PG 实例，最后再清空$PGDATA（对于目标端非新环境等场景）

```
shell> su – postgres
shell> echo $PGDATA
/var/lib/postgresql/14/main
shell> cd $PGDATA
shell> ls -l

shell> pg_ctl -D $PGDATA stop
shell> pg_ctl -D $PGDATA status
shell> rm -rf /var/lib/postgresql/14/main/*
```

### 2、备库远程拷贝主库数据目录

```
shell> pg_basebackup -h 10.211.55.4 -U repl -D /postgres/product/data/ -X stream -P -R
Password: 
33064/33064 kB (100%), 1/1 tablespace
```

在/postgres/product/data 下，会比主库多出一个 backup_label 文件，该文件记录了备份开始时 WAL 日志位置 ，如下：

```
shell> more $PGDATA/backup_label
START WAL LOCATION: 0/2000028 (file 000000010000000000000002)
CHECKPOINT LOCATION: 0/2000060
BACKUP METHOD: streamed
BACKUP FROM: primary
START TIME: 2022-12-05 22:48:40 HKT
LABEL: pg_basebackup base backup
START TIMELINE: 1
```

3、配置 postgresql.auto.conf

注意：如果 pg_basebackup 时添加了-R 选项，则可跳过此步，否则必须配置此步骤。
以下为没有添加-R 选项的配置方法，如下：

```
shell> vi $PGDATA/postgresql.auto.conf
primary_conninfo = 'host=10.211.55.4 port=5432 user=repl password=Abcd321#'
recovery_target_timeline = 'latest'
#recovery_min_apply_delay=5ms #应用延迟，有特殊需求可以设置
```

手动创建 standby.signal 文件

```
shell> touch $PGDATA/standby.signal
```

### 4、启动备库实例

```
shell> pg_ctl -D $PGDATA start
```

使用systemctl命令

```
(root)shell> systemctl start postgresql
```

## 验证主备关系

### 1、检查主库与备库角色

在主库执行角色检查，如下：
```
shell> pg_controldata | grep 'Database cluster state'
Database cluster state: in production
```

在备库执行角色检查，如下：
```
shell> pg_controldata | grep 'Database cluster state'
Database cluster state: in archive recovery
```

### 2、在主库上检查同步情况

```
postgres=# \x
Expanded display is on.
postgres=# select pid,usename,application_name,client_addr,state,sync_state from pg_stat_replication;
-[ RECORD 1 ]----+------------
pid              | 8470
usename          | repl
application_name | 14/main
client_addr      | 10.211.55.4
state            | streaming
sync_state       | async
```

### 3、在备库检查同步情况

```
postgres=# \x
Expanded display is on.
postgres=# select * from pg_stat_wal_receiver;
-[ RECORD 1 ]---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
pid                   | 94119
status                | streaming
receive_start_lsn     | 0/4000000
receive_start_tli     | 1
written_lsn           | 0/5000148
flushed_lsn           | 0/5000148
received_tli          | 1
last_msg_send_time    | 2022-12-05 23:50:29.927646+08
last_msg_receipt_time | 2022-12-05 23:50:29.755814+08
latest_end_lsn        | 0/5000148
latest_end_time       | 2022-12-05 23:33:28.264379+08
slot_name             |
sender_host           | 10.211.55.5
sender_port           | 5432
conninfo              | user=repl password=******** channel_binding=prefer dbname=replication host=10.211.55.5 port=5432 fallback_application_name=14/main sslmode=prefer sslcompression=0 sslsni=1 ssl_min_protocol_version=TLSv1.2 gssencmode=prefer krbsrvname=postgres target_session_attrs=any
```

