---

title: Python 3.11 源码安装文档
date: 2025-08-12
tags: [ 技术文档 ]

---

本文档基于CentOS 7.9环境，介绍如何源码编译安装Python 3.11.2。

## 安装步骤

### 1. 配置软件源

```bash
# 配置华为云镜像源（可选，提高下载速度）
curl -o /etc/yum.repos.d/CentOS-Base.repo https://repo.huaweicloud.com/repository/conf/CentOS-7-reg.repo
yum clean all
rm -rf /var/cache/yum
yum makecache
```

### 2. 安装编译依赖

```bash
# 安装EPEL源
yum install -y epel-release

# 安装编译Python所需的依赖包
yum install -y \
    gcc \
    zlib zlib-devel \
    libffi libffi-devel \
    readline-devel \
    make \
    openssl-devel openssl11 openssl11-devel \
    sqlite-devel \
    bzip2-devel \
    libuuid-devel \
    xz-devel \
    tk-devel \
    gdbm-devel
```

### 3. 下载Python源码

```bash
# 创建临时目录
mkdir -p /tmp/python-build
cd /tmp/python-build

# 下载Python 3.11.2源码（优先使用华为云镜像）
curl -fSL https://mirrors.huaweicloud.com/python/3.11.2/Python-3.11.2.tgz -o python.tgz || \
    curl -fSL https://www.python.org/ftp/python/3.11.2/Python-3.11.2.tgz -o python.tgz

# 解压源码
tar -zxf python.tgz
cd Python-3.11.2
```

### 4. 配置编译选项

这里指定Python安装路径为 `/usr/python/`

```bash
# 设置安装目录
export PREFIX=/usr/python

# 配置OpenSSL环境变量（支持OpenSSL 1.1）
export CPPFLAGS=-I/usr/include/openssl11
export LDFLAGS=-L/usr/lib64/openssl11

# 配置编译选项
./configure --prefix=${PREFIX} \
    --enable-loadable-sqlite-extensions \
    --enable-option-checking=fatal \
    --enable-shared \
    --with-lto \
    --with-system-expat
```

#### 配置选项说明：

- `--prefix=${PREFIX}`: 指定安装目录为/usr/python
- `--enable-loadable-sqlite-extensions`: 启用SQLite扩展支持
- `--enable-option-checking=fatal`: 严格检查配置选项
- `--enable-shared`: 生成共享库
- `--with-lto`: 启用链接时优化
- `--with-system-expat`: 使用系统的expat库

### 5. 编译安装

```bash
# 编译（使用所有CPU核心加速）
make -j "$(nproc)" \
    "EXTRA_CFLAGS=$(rpm --eval '%{optflags}')" \
    "LDFLAGS=-Wl,--strip-all"

# 重新编译python可执行文件（设置运行时库路径）
rm -rf python
make -j "$(nproc)" \
    "EXTRA_CFLAGS=$(rpm --eval '%{optflags}')" \
    "LDFLAGS=-Wl,-rpath='\$\$ORIGIN/../lib'" \
    python

# 安装Python
make install
```

### 6. 清理安装文件（可选）

```bash
# 删除测试文件和不必要的文件以减小安装体积
find ${PREFIX} -depth \
    \( \
        \( -type d -a \( -name test -o -name tests -o -name idle_test \) \) \
        -o \( -type f -a \( -name '*.pyc' -o -name '*.pyo' -o -name 'libpython*.a' \) \) \
    \) -exec rm -rf '{}' +
```

### 7. 配置环境变量

```bash
# 更新动态链接库缓存
ldconfig

# 设置环境变量
echo 'export LD_LIBRARY_PATH="/usr/python/lib:$LD_LIBRARY_PATH"' >> ~/.bashrc
echo 'export PATH="/usr/python/bin:$PATH"' >> ~/.bashrc

# 立即生效
export LD_LIBRARY_PATH="/usr/python/lib:$LD_LIBRARY_PATH"
export PATH="/usr/python/bin:$PATH"
```

### 8. 创建系统链接（可选）

```bash
# 创建python和pip的系统链接
ln -sf /usr/python/bin/python3 /usr/bin/python
ln -sf /usr/python/bin/pip3 /usr/bin/pip

# 注意：如果系统依赖python2，需要修改相关脚本
sed -i '1s/^.*$/#!\/usr\/bin\/python2.7/' /usr/bin/yum
sed -i '1s/^.*$/#!\/usr\/bin\/python2.7/' /usr/libexec/urlgrabber-ext-down
```

### 9. 验证安装

```bash
# 检查Python版本
python --version
# 应该输出：Python 3.11.2

# 检查pip版本
pip --version

# 测试Python功能
python -c "import ssl; print('SSL support:', ssl.OPENSSL_VERSION)"
python -c "import sqlite3; print('SQLite version:', sqlite3.sqlite_version)"
```

## 清理工作

```bash
# 清理编译依赖（可选）
yum -q -y remove epel-release
yum -q -y autoremove
yum -q -y clean all
rm -rf /var/cache/yum/*

# 删除源码目录
rm -rf /tmp/python-build
```