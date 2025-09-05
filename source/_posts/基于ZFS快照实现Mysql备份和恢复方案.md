---

title: 基于ZFS快照实现Mysql备份和恢复方案
date: 2025-09-05
tags: [ 技术文档 ]

---

演示Mysql的数据目录存在路径：`/u02/mysql84/app/mysqldb`

### 现有文件系统上创建一个文件作为 ZFS 池：

无多余的磁盘，这里仅作验证快照备份和恢复功能实例。

```bash
# 创建存储目录
mkdir -p /opt/zfs

# 创建一个 20GB 的存储文件 (可以根据需要调整大小)
dd if=/dev/zero of=/opt/zfs/mysql_pool.img bs=1G count=20 status=progress

# 检查创建的文件
ls -lh /opt/zfs/mysql_pool.img

# 创建 ZFS 存储池
zpool create mysql_pool /opt/zfs/mysql_pool.img

# 检查存储池状态
zpool status mysql_pool

# 查看存储池列表
zpool list
```

### 创建 ZFS 数据集并配置优化参数:

```
# 创建 MySQL 数据集，配置为 MySQL 数据库优化
zfs create mysql_pool/mysqldb
# 设置压缩（使用 lz4 算法，性能好）
zfs set compression=lz4 mysql_pool/mysqldb
# 设置记录大小（对 MySQL 优化）
zfs set recordsize=16K mysql_pool/mysqldb
# 启用数据去重（可选，视数据特征而定）
zfs set dedup=off mysql_pool/mysqldb
# 设置访问时间更新（性能优化）
zfs set atime=off mysql_pool/mysqldb
# 设置同步策略（平衡性能和安全）
zfs set sync=standard mysql_pool/mysqldb

```

### 迁移 MySQL 数据：

使用临时挂载点 `/u02/mysql84/app/mysqldb_new` ，避免挂载失败。

```bash

# 设置临时挂载点
zfs set mountpoint=/u02/mysql84/app/mysqldb_new mysql_pool/mysqldb

# 设置正确的权限和所有者
chown mysql:mysql /u02/mysql84/app/mysqldb_new

# 复制数据到新的 ZFS 数据集
cp -rp /u02/mysql84/app/mysqldb/* /u02/mysql84/app/mysqldb_new/

# 验证数据完整性
diff -r /u02/mysql84/app/mysqldb /u02/mysql84/app/mysqldb_new || echo 'Some differences found, please review'

# 卸载当前的 ZFS 数据集
zfs umount mysql_pool/mysqldb

# 重新设置挂载点为原路径
zfs set mountpoint=/u02/mysql84/app/mysqldb mysql_pool/mysqldb

# 挂载数据集
zfs mount mysql_pool/mysqldb

# 检查最终结果
ls -la /u02/mysql84/app/mysqldb/
zfs list mysql_pool/mysqldb
rmdir /u02/mysql84/app/mysqldb_new

# 检查压缩效果
zfs get compressratio mysql_pool/mysqldb
```

**迁移完成后要重启Mysql数据库**

### 创建快照 ：

```
# 直接创建快照
zfs snapshot mysql_pool/mysqldb@20250905_1447

# 查看创建的快照
zfs list -t snapshot

# 设置定时任务 - 每4小时创建一次快照
echo '0 */4 * * * root /usr/sbin/zfs snapshot mysql_pool/mysqldb@auto_$(date +\%Y\%m\%d_\%H\%M\%S)' >> /etc/crontab

# 显示 crontab
tail -1 /etc/crontab
```

### 日常管理命令 :

```
# 查看储池状态
zpool status mysql_pool

# 查看数据集信息
zfs list mysql_pool/mysqldb

# 查看压缩效果
zfs get compressratio mysql_pool/mysqldb

# 查看所有快照
zfs list -t snapshot

# 手动创建快照
zfs snapshot mysql_pool/mysqldb@manual_$(date +%Y%m%d_%H%M%S)

# 恢复到快照 (需要停止 MySQL)
zfs rollback mysql_pool/mysqldb@snapshot_name

# 删除快照
zfs destroy mysql_pool/mysqldb@snapshot_name
```


## 验证快照恢复功能 ：

验证思路是：1. 创建临时数据库并插入初始化数据，2. 做快照tag1，3. 再插入新的数据，4. 快照tag1恢复。会发现步骤3插入的数据已经被遗弃了。

### 创建测试数据库和表 ，并插入数据：

```sql
-- 创建 testdb 数据库
CREATE DATABASE IF NOT EXISTS testdb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

USE testdb;

-- 创建 user 表
CREATE TABLE IF NOT EXISTS user (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- 插入 id=1 的数据
INSERT INTO user (id, username, email) VALUES (1, "test_user", "test@example.com");
```

### 手动创建快照 ：

```
zfs snapshot mysql_pool/mysqldb@test_initdb

# 查看快照
zfs list -t snapshot
```

### 继续插入测试数据 ：

```sql
USE testdb;
INSERT INTO user (id, username, email) VALUES (2, "test_user2", "test2@example.com");

-- 查询表数据能看到两条数据
SELECT * FROM user;
```

### 快照恢复:

该步骤需要启停数据库。

```bash

# 停止数据库

# 快照恢复
zfs rollback mysql_pool/mysqldb@test_initdb

# 启动数据库

```

### 查看恢复的数据 ：

```sql
USE testdb;
-- 这里只有一条数据
select * from user;
```