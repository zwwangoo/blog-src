---
title: MySQL服务器配置远程数据库访问
date: 2016-10-18
tags: [数据库, Linux]
---

## 安装Mysql，并配置远程访问

### 更新源

    sudo apt-get update


### 安装mysql

    sudo apt-get install mysql-server mysql-client

软件包设置对话框"中输入mysql中"root"用户的密码以及确认密码。

注意：

安装过程没有提示设置密码，那么完成之后，可以在`/etc/mysql/debian.cnf` 有着MySQL默认的用户名和用户密码， 用户名默认的不是root，而是debian-sys-maint

密码会随即给一个很复杂的，这个时候，要进入MySQL的话，就是需要在终端把root更改为debian-sys-maint

所安装的版本是5.7，所以password字段已经被删除，取而代之的是`authentication_string`字段，所以要更改密码：

```
mysql> use mysql;

mysql> update user set authentication_string=PASSWORD("这里输入你要改的密码") where User='root';  # 更改密码
mysql> update user set plugin="mysql_native_password";  # 如果没这一行可能也会报一个错误，因此需要运行这一行

mysql> flush privileges;  # 更新所有操作权限
mysql> quit;
```

<!--more-->

### 判断mysql是否安装成功

    sudo service mysql restart

如果mysql启动成功，处于运行状态说明mysql安装成功。

### 查看mysql版本

    mysql --version

### 登录mysql

    mysql -u root -p

输入mysql中"root"用户的密码

说明：

-P: 表示服务端口,有些时候Mysql提供服务的端口改成其他端口。建议是改成其他端口，比如13306

### 创建远程登录用户

    CREATE USER 'username'@'host' IDENTIFIED BY 'password';

说明:username - 你将创建的用户名, host - 指定该用户在哪个主机上可以登陆,如果是本地用户可用localhost, 如果想让该用户可以从任意远程主机登陆,可以使用通配符%. password - 该用户的登陆密码,密码可以为空,如果为空则该用户可以不需要密码登陆服务器.


例子:

```
CREATE USER 'dog'@'localhost' IDENTIFIED BY '123456';  
CREATE USER 'pig'@'192.168.1.101_' IDENDIFIED BY '123456';  
CREATE USER 'pig'@'%' IDENTIFIED BY '123456';  
CREATE USER 'pig'@'%' IDENTIFIED BY '';  
CREATE USER 'pig'@'%';
```

### 授权:

    GRANT privileges ON databasename.tablename TO 'username'@'host'；

说明: privileges - 用户的操作权限,如SELECT , INSERT , UPDATE 等(详细列表见该文最后面).如果要授予所的权限则使用ALL.;databasename - 数据库名,tablename-表名,如果要授予该用户对所有数据库和表的相应操作权限则可用*表示, 如*.*.


例子：

```
GRANT SELECT, INSERT ON test.user TO 'pig'@'%';  
GRANT ALL ON *.* TO 'pig'@'%';
```

注意:用以上命令授权的用户不能给其它用户授权,如果想让该用户可以授权,用以下命令:  

    GRANT privileges ON databasename.tablename TO 'username'@'host' WITH GRANT OPTION;

授权完成之后：

    flush privileges;  # 更新所有操作权限

### 修改监听ip

~~ 在目录/etc/mysql下找到my.cnf，用vim编辑，找到my.cnf里面的 ~~
注：mysql 5.5版本，修改`/etc/my.cnf`，mysql 5.7 修改 `/etc/mysql/mysql.conf.d/mysqld.cnf` 文件。

    bind-address           = 127.0.0.1

将其注释。

### 重启数据库服务器

	sudo service mysql restart

## MySQL的一些其他配置

### mysql设置中文输入

mysql 5.5版本，修改`/etc/my.cnf`，mysql 5.7 修改 `/etc/mysql/mysql.conf.d/mysqld.cnf` 文件。

添加以下内容：

```
[client]
default-character-set=utf8
[mysqld]
default-storage-engine=INNODB
character-set-server=utf8
collation-server=utf8_general_ci
```

重启mysql(`sudo service mysql restart`)

进入mysql查看字符集(mysql>`show variables like 'character_set_%';`)

### 忽略表名大小写

在配置文件中添加以下内容：

```
[mysqld]
lower_case_table_names=1 # 1是不区分，0是区分
```

重启mysql(`sudo service mysql restart`)

### 说明

mysql默认端口号为3306
不建议使用root帐号做远程登录的帐号

---

## 安装postgresql，并配置远程用户访问

### 更新源列表

    sudo apt-get update

### 安装postgresql

    sudo apt-get install postgresql

系统会提示安装所需磁盘空间，输入"y"，安装程序会自动完成。 安装完毕后，系统会创建一个数据库超级用户“postgres”, 密码为空。这个用户既是不可登录的操作系统用户，也是数据库用户。


### 修改Linux用户postgres的密码

    sudo passwd postgres

### 修改数据库超级用户postgres的密码

#### 1 切换到Linux下postgres用户

    sudo su postgres

#### 2 登录postgres数据库

    psql postgres

这样你就可以看到一下提示信息：

```
psql (8.4.4)
Type "help" for help.
```

并出现postgres的命令提示符号：

    postgres=#

#### `ALTER USER postgres with PASSWORD 'password';`, 键入`q`返回到Linux命令行。

### 设置其它机器上对postgres的访问

修改

```
/etc/postgresql/8.4/main/pg_hba.conf:
host all all 0.0.0.0/0 md5  #0.0.0.0
```

为地址段，0为多少二进制位
例如：192.168.0.0/16代表192.168.0.1-192.168.255.254
修改/etc/postgresql/8.4/main/postgresql.conf

    listen_address = '*'

### 重启数据库

    sudo /etc/init.d/postgresql restart

### 说明

postgresql的默认端口号为5432


注：配置完成之后，别忘了将服务器主机的防火墙关闭这些端口的防护
