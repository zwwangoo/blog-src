---
title: Mysql主从复制搭建
date: 2024-04-07
tags: [数据库, mysql]
---


1 备份源库

```
./bin/mysqldump -u root -pmysql --single-transaction -A -R -E --triggers --master-data=2 --flush-logs > full.sql
```

备份完成后，记录文件中 `CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=154;`



2 修改源和目标的mysql配置（service-id 主从不能相同）：



```
[mysqld]

server-id = 11
log-bin = mysql-bin
binlog_format = ROW
```



3 重启主从

```
service mysqld restart
```

4 生产端创建复制用户

```mysql
create user repl@'%' identified by 'repl@admin!123';
grant REPLICATION SLAVE on *.* to 'repl'@'%';
flush privileges;
```

5 目标端全局恢复

6 目标端配置复制连接

MySQL8.0默认使用caching_sha2_password身份验证插件需要指定get_master_public_key=1

```mysql
change master to master_host='192.168.2.181',master_user='repl',master_password='repl@admin!123',master_port=3306,MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=154;
```

7 启动复制

```
start slave
```

8 查看复制状态

```
show slave status \G
```

输出：



```
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.2.181
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 154
               Relay_Log_File: debian11-181-relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
            ...
```

则主从复制搭建完毕
