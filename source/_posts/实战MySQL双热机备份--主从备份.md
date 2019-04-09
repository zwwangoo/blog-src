---
title: 实战MySQL双热机备份--主从备份
date: 2019-04-09
tags: [数据库, Linux]
---

经过惊心动魄的两个小时，终于完成了对约车系统的数据库热备份(主从备份)。其实来说热备份并不难实现，在本地模拟的时候，不到十分钟就解决了，但是生产环境上的MySQL版本为5.5，从MySQL版本为5.7 不同的版本之间出了一些问题，不得不升级生产环境上的MySQL，然后的然后，收获最大的经验就是：在生产环境上，不管干什么操作，多备份几份数据库，多在不同的地方备份数据库！！可能某个误操作，让你惊了一身冷汗！但是还好，我有多个备份！！

后来冷静的发现，其实备份丢失了也没有什么大的问题，毕竟机智的我开启了log-bin，有我需要的日志就可以了。

下面记录一下本次实战MySQL双机热备份的过程，供以后参考。

参考：

- [学一点 mysql 双机异地热备份----快速理解mysql主从，主主备份原理及实践](https://www.cnblogs.com/shuidao/p/3551238.html)
- [Mysql双机热备实现](https://blog.csdn.net/zheng963/article/details/72385123)
- [利用 MySql日志文件 恢复数据](https://blog.csdn.net/dbanote/article/details/39692219)
- [Ubuntu 14.04升级mysql到5.7](https://www.typechodev.com/case/754.html)

<!--more-->

## 环境说明

A 服务器（主服务器master）：192.168.121.122  Ubuntu16.04 Server
B 服务器（从服务器slave）： 192.168.121.108  Ubuntu16.04 Server

MySQL版本皆为5.7.25，查看当前MySQL版本：

```bash
mysql --version
```

将主服务器需要同步的数据库内容进行备份一份，上传到从服务器上，保证始初时两服务器中数据库内容一致。

##  在服务器开启远程登录

### 1 授权用户可以远程访问

mysql -u root -p输入密码进行登录，

之后执行命令：

```mysql
grant all privileges on *.* to 'root'@'%' identified by 'password';
```

对该条命令的说明：

- 第一个\*是数据库，可以改成允许访问的数据库名称
- 第二个\* 是数据库的表名称，\*代表允许访问任意的表
- root代表远程登录使用的用户名，可以自定义
- %代表允许任意ip登录，如果你想指定特定的IP，可以把%替换掉就可以了
- password代表远程登录时使用的密码，可以自定义

让权限立即生效, 执行

```mysql
flush privileges;
```

如果不希望使用root用户进行远程登录请参考[服务器配置远程数据库访问](https://suadminwen.github.io/2016/10/18/%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%85%8D%E7%BD%AE%E8%BF%9C%E7%A8%8B%E6%95%B0%E6%8D%AE%E5%BA%93%E8%AE%BF%E9%97%AE/)


### 2 修改配置文件

mysql 5.5版本，修改/etc/my.cnf，mysql 5.7 修改 /etc/mysql/mysql.conf.d/mysqld.cnf 文件，编辑该文件，找到bind-address = 127.0.0.1这一句，然后在前面加个#号注释掉，保存退出；

### 3 重启服务

```
sudo service mysql restart
```

## 主服务器master的配置

开启远程登录：
账号 root
密码 123456。

### 修改配置文件

修改 /etc/mysql/mysql.conf.d/mysqld.cnf 文件，在[mysqld]下修改即可：

```
[mysqld]
server-id = 1
log-bin=/var/log/mysql/mysql-bin.log  //其中这两行是本来就有的，可以不用动，添加下面两行即可

binlog-do-db = busbook
binlog-ignore-db = mysql  // 这行可以不用写
```

修改说明

- binlog-do-db 用来表示，只把哪些数据库的改动记录到binary日志中。 可以写多行，表示关注多个数据库。
- binlog-ignore-db 表示，需要忽略哪些数据库。

### 重启服务器

修改完配置文件后，保存后，重启一下mysql服务，如果成功则没问题。

### 查看主服务器状态

为了保证操作过程中没有数据变化，需要锁住当前的数据库，登录MySQL服务后，通过命令：

```
flush tables with read lock;
```

然后查看状态：

```
show master status\G
```
这里给出实例输出，因为这个输出很重要：

```
mysql> show master status\G
*************************** 1. row ***************************
             File: mysql-bin.000002
         Position: 154
     Binlog_Do_DB: busbook
 Binlog_Ignore_DB: 
Executed_Gtid_Set: 
1 row in set (0.00 sec)
```
**记住这里的输出关键`File`和`Position`的值，等会在从服务器的配置中会用到。**

主服务器已经做完了， 可以解除锁定了:

```
ublock tables;
```

## 从服务器slave的配置

我这里只做了主从服务器的备份，没有做主主服务器的备份，所以只需要完成如下操作即可。

### 修改配置文件

修改 /etc/mysql/mysql.conf.d/mysqld.cnf 文件，在[mysqld]下修改即可：

```
[mysqld]
server-id = 2
log_bin = /var/log/mysql/mysql-bin.log

replicate-do-db   = busbook
# replicate-ignore-db = mysql
relay_log 		  = /var/log/mysql/mysql-relay-bin.log
log-slave-updates = ON
```

修改说明:

- server-id 必须保证每个服务器不一样。 这可能和循环同步有关。 防止进入死循环。
- replicate-do-db 可以指定需要复制的数据库。
- replicate-ignore-db 复制时需要排除的数据库。
- relay_log 中继日志的名字。复制线程需要先把远程的变化拷贝到这个中继日志中， 再执行。
- log-slave-updates 意思是，中继日志执行之后，这些变化是否需要计入自己的binarylog。 当你的B服务器需要作为另外一个服务器的主服务器的时候需要打开。  就是双主互相备份，或者多主循环备份。 

### 重启服务器

修改完配置文件后，保存后，重启一下mysql服务，如果成功则没问题。

### 用change master 语句指定同步位置

这步是最关键的一步了，在进入mysql操作界面后，输入如下指令：

```
stop slave;
```

先停步slave服务线程，这个是很重要的，如果不这样做会造成以下操作不成功。

```
mysql> change master to
> master_host='192.168.121.122',master_user='root',master_password='123456',
> master_log_file=' mysql-bin.000002',master_log_pos=154;
```
说明：
- master_log_file, master_log_pos由主服务器（Master）查出的状态值中确定。 master_log_file对应File, master_log_pos对应Position。

则可以开启slave线程了：

```
start slave;
```

### 查看从服务器（Slave）状态

查看状态：

```
show slave status\G
```

实例输出：

```
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.121.122
                  Master_User: root
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 154
               Relay_Log_File: mysqld-relay-bin.000006
                Relay_Log_Pos: 367
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: busbook
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 154
              Relay_Log_Space: 741
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
                  Master_UUID: 9198b1a6-58db-11e9-9f47-08002777fdc0
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)

```

如果看到以下两行值均为YES，则表示设置从服务器成功。

```
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```
如果其中一个是No， 那就说明不成功。需要查看mysql的错误日志。有时候密码填错了， 有时候防火墙的3306没有打开。ip地址不对，等等。 都会导致失败。错误日志一般在/var/log/mysql/error.log，是否进行修改过，需要查看配置文件中的具体配置。

到此，基本上完成了MySL主从备份的过程，可以进行测试，改变主服务器的数据，查看从服务器时，发现已经同步发生改变，即为成功。
