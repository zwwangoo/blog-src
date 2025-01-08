---

title: Centos换源
date: 2024-08-14
tags: [ 技术文档 ]

---

## Centos7换源

- 备份一下当前的源（`/etc/yum.repos.d/CentOS-Base.repo`），以防出错后可以还原回来

```
cp /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
```

- CentOS 旧版本的软件包和已不再维护的 CentOS 版本都会被存储到 http://vault.centos.org，所以只需要将 repo 文件的 baseurl 由 http://mirrorlist.centos.org 改为 http://vault.centos.org 即可，可以使用如下命令：
```
sed -i s/mirror.centos.org/vault.centos.org/g /etc/yum.repos.d/*.repo
sed -i s/^#.*baseurl=http/baseurl=http/g /etc/yum.repos.d/*.repo
sed -i s/^mirrorlist=http/#mirrorlist=http/g /etc/yum.repos.d/*.repo
```

- 生成源缓存并更新yum源

```
yum makecache
yum update
```

- centos7 yum 安装 openssl 1.1.1

```
yum install -y epel-release 
yum install -y openssl11 openssl11-devel
ln -sf /usr/lib64/pkgconfig/openssl11.pc /usr/lib64/pkgconfig/openssl.pc
```

## Centos8 Stream 换源

- 使用阿里云源

```
sudo sed -e 's|^mirrorlist=|#mirrorlist=|g' \
         -e 's|^#baseurl=http://mirror.centos.org/$contentdir|baseurl=https://mirrors.aliyun.com/centos|g' \
         -i.bak \
         /etc/yum.repos.d/CentOS-AppStream.repo \
         /etc/yum.repos.d/CentOS-BaseOS.repo \
         /etc/yum.repos.d/CentOS-Extras.repo \
         /etc/yum.repos.d/CentOS-Stream-PowerTools.repo
```

- 生成源缓存并更新yum源

```
yum makecache
yum update
```
