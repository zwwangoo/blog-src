---

title: InterSystems Cache数据库常用命令
date: 2025-01-06
tags: [ 数据库 ]

---

## Docker 运行 Cache：

无 License 版本存在并发会话数最多为 1 的限制

```
docker pull daimor/intersystems-cache:2018.1
docker run -d -p 1972:1972 -p 57772:57772 daimor/intersystems-cache:2018.1
```

-  Cache 的管理页面: [http://127.0.0.1:57772/csp/sys/UtilHome.csp](http://127.0.0.1:57772/csp/sys/UtilHome.csp)

## 终端使用

查看实例信息：

```bash
$ ccontrol all
    Instance Name     Version ID        Port   Directory
    ----------------  ----------------  -----  --------------------------------
up >CACHE             2018.1.5.659.0    1972   /usr/cachesys/
```

常用命令：

- 查看当前实例信息： `ccontrol all`
- 启动 ISCAgent：`/etc/init.d/ISCAgent start`
- 登录 terminal： `csession cache`
- 切换命名空间： `zn "%sys"`
- 进入 SQL Shell：`do $SYSTEM.SQL.Shell()`
- 退出数据库：`halt`

以下示例创建用户并赋权：

```bash
$ cache
USER> zn "%sys"

%SYS> do $SYSTEM.SQL.Shell()
SQL Command Line Shell
----------------------------------------------------

The command prefix is currently set to: <<nothing>>.
Enter q to quit, ? for help.
%SYS>> 

# 创建用户
%SYS>> CREATE USER NewUser IDENTIFIED BY 'NewPassword'
1.      CREATE USER NewUser IDENTIFIED BY 'NewPassword'

0 Rows Affected
statement prepare time(s)/globals/lines/disk: 0.0073s/1654/11280/0ms
          execute time(s)/globals/lines/disk: 0.0035s/173/1817/0ms
                          cached query class: %sqlcq.pSYS.cls6
---------------------------------------------------------------------------
%SYS>> q

# 查询新用户是否存在，注意已经从sql shell退出了，输出1则表示存在
%SYS>WRITE ##class(Security.Users).Exists("NewUser")
1
%SYS>

# 为用户分配 %ALL 角色
%SYS> DO ##class(Security.Users).AddRoles("NewUser", "%ALL")

# 退出

%SYS> halt
```

## Python 驱动连接（一般路径为 `/usr/cachesys/dev/python` ）：

```python
import intersys.pythonbind3

# 资产信息
ip = '127.0.0.1'
port = '1972'
namespace = '%SYS'
user = 'NewUser'
password = 'NewPassword'
timeout = 5

# 建立连接
conn = intersys.pythonbind3.connection()
url = f'{ip}[{port}]:{namespace}'
conn.connect_now(url, user, password, timeout)

# 查询
database = intersys.pythonbind3.database(conn)

# 查询数据库版本号，执行类方法
database.run_class_method('%SYSTEM.Version', 'GetVersion', [])


# 普通查询语句
sqlstring ="SELECT ID, Name, DOB, SSN \
            FROM SAMPLE.PERSON \
            WHERE Name %STARTSWITH ?"
query = intersys.pythonbind3.query(database)
query.prepare(sqlstring)
query.set_par(1,"A")
query.execute();
while True:
   cols = query.fetch([None])
   if len(cols) == 0: break
   print(cols)
```

## Python 驱动打包

### 容器内换源

```bash
sed -i s/mirror.centos.org/vault.centos.org/g /etc/yum.repos.d/*.repo
sed -i s/^#.*baseurl=http/baseurl=http/g /etc/yum.repos.d/*.repo
sed -i s/^mirrorlist=http/#mirrorlist=http/g /etc/yum.repos.d/*.repo

yum makecache
yum update -y
```

### 安装依赖

```bash
yum install -y gcc gcc-c++ python3 python3-devel
```

### 安装

注意 `/usr/cachesys/bin` 是驱动

```python
cd /usr/cachesys/dev/python
python3 setup3.py install <<EOF
> /usr/cachesys
> EOF

# 要从上述install的目录跳出来
cd /usr/cachesys
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/cachesys/bin
pip3 show pythonbind3
```

### 打包

这里示例打包为 wheel 包，需要安装孔 wheel，如果 Python 版本升级或修改包名，需要修改 `setup3.py` 中相关配置。

```bash
cd /usr/cachesys/dev/python
pip3 install wheel
python3 setup3.py bdist_wheel <<EOF
> /usr/cachesys
> EOF
ll dist/pythonbind3-1.0-cp36-cp36m-linux_x86_64.whl

```

其他环境安装该包，需要 copy 相关的驱动依赖 `/usr/cachesys/bin`