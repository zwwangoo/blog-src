---

title: Linux 安装Nginx
date: 2024-10-14
tags: [ 技术文档 ]

---
0、安装依赖
```
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
```

1、下载并解压到 `/usr/local/nginx-1.13.7`

```
curl -LO http://nginx.org/download/nginx-1.13.7.tar.gz
tar -zxvf nginx-1.13.7.tar.gz -C /usr/local/
cd /usr/local/nginx-1.13.7/
```

2、安装
```
./configure
# 执行make命令
make
# 执行make install命令
make install
```

3、修改端口配置文件路径 `/usr/local/nginx/conf/nginx.conf`

4、启动
```
/usr/local/nginx/sbin/nginx
# 停止
/usr/local/nginx/sbin/nginx -s stop
```

5、检查

```
ps aux | grep nginx
```