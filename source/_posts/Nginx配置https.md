---
title: Nginx配置https
date: 2023-11-03
tags: [Linux, Nginx]
---

1、阿里云SSL证书申请

（1）进入免费证书控制台: https://yundun.console.aliyun.com/?p=cas#/certExtend/free

（2）点击证书申请，添加信息

<img src="/blog-img/2f437332258223c84eb08b100008ac56.png" width="583" height="433">

(3) 提交审核通过后，在列表界面点击下载;上传证书到服务器，我的证书存放地址在/etc/nginx/cert/中

2、在 /etc/nginx/sites-enabled/ 目录下创建 对应的域名配置，我这里配置的是gitea.conf，复制一下内容并修改对应的域名和证书名称；

```nginx
server {
    listen       443 ssl;

    server_name gitea.wangzhiwen.top;

    server_name_in_redirect off;
    proxy_set_header Host $host:$server_port;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header REMOTE-HOST $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    ssl_certificate /etc/nginx/cert/8682750_gitea.wangzhiwen.top.pem;
    ssl_certificate_key /etc/nginx/cert/8682750_gitea.wangzhiwen.top.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    #表示使用的加密套件的类型。
    ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3; #表示使用的TLS协议的类型，您需要自行评估是否配置TLSv1.1协议。
    ssl_prefer_server_ciphers on;

    location / {
        proxy_pass http://localhost:3000;
        proxy_redirect default;
    }
}

server {
    listen       80;
    server_name  gitea.wangzhiwen.top;
    return 301 https://$server_name$request_uri;
}
```

配置文件中有两个server，第一个为监听的443端口https服务，第二个为监听80端口，将http请求转发到https。

nginx配置说明：

- `listen` 监听的端口
- `server_name` 监听的域名
- `proxy_set_header` Nginx代理转发的header信息
- `ssl_certificate` pem文件路径
- `ssl_certificate_key` key文件路径
- `location` 地址定向，数据缓存，应答控制，以及第三方模块的配置

