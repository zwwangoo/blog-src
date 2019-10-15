---
title: 自建V2Ray和加速
date: 2019-10-13
tags: [VPN, 服务器]
---


购买vps服务，这里推荐 [digitalocean](https://m.do.co/c/9491e366d1c5)（如果没有使用过的朋友可以通过我的邀请链接注册。）。目前来说比较稳定，系统最好选择Centos，创建服务之后，先ping一下分配的ip，如果ping不通，则删掉换区重建。

## 自动安装v2ray工具：

```
wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh" && chmod +x tcp.sh && ./tcp.sh
```

## 安装BBR：

```
wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh" && chmod +x tcp.sh && ./tcp.sh
```

先选择安装安装 BBR/BBR魔改版内核内核，安装完成后，
重启系统后再运行tcp.sh按需要选择加速，我选择的是魔改版加速。

## Linux下载客户端脚本

```
wget https://install.direct/go.sh
sudo base go.sh
```
自动安装的路径：

- /usr/bin/v2ray/v2ray：V2Ray 程序；
- /usr/bin/v2ray/v2ctl：V2Ray 工具；
- /etc/v2ray/config.json：配置文件；
- /usr/bin/v2ray/geoip.dat：IP 数据文件
- /usr/bin/v2ray/geosite.dat：域名数据文件

会在系统重启之后，自动运行 V2Ray。之后需要编辑 /etc/v2ray/config.json 文件来配置你需要的代理方式；
运行 `service v2ray start` 来启动 V2Ray 进程；

## v2ray客户端配置 ` config.json`

```
{
  "log": {
    "access": "",
    "error": "",
    "loglevel": "warning"
  },
  "inbound": {
    "port": 1080, // SOCKS 代理端口，在浏览器中需配置代理并指向这个端口
    "listen": "127.0.0.1",
    "protocol": "socks",
    "domainOverride": [
      "tls",
      "http"
    ],
    "settings": {
      "auth": "noauth",
      "udp": true,
      "ip": "127.0.0.1",
      "clients": null
    },
    "streamSettings": null
  },
  "outbound": {
    "tag": "agentout",
    "protocol": "vmess",
    "settings": {
      "vnext": [
        {
          "address": "159.65.xxx.xxx", // 服务器地址，请修改为你自己的服务器 ip 或域名
          "port": 47297,  // 服务器端口
          "users": [
            {
              "id": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx", //客户端和服务器统一的ID号
              "alterId": 233
            }
          ]
        }
      ],
      "servers": null
    },
    "streamSettings": {
      "network": "tcp",
      "security": "",
      "tlsSettings": null,
      "tcpSettings": null,
      "kcpSettings": null,
      "wsSettings": null,
      "httpSettings": null
    },
    "mux": {
      "enabled": true
    }
  },
  "inboundDetour": null,
  "outboundDetour": [
    {
      "protocol": "freedom",
      "settings": {
        "response": null
      },
      "tag": "direct"
    },
    {
      "protocol": "blackhole",
      "settings": {
        "response": {
          "type": "http"
        }
      },
      "tag": "blockout"
    }
  ],
  "dns": {
    "servers": [
      "8.8.8.8",
      "8.8.4.4",
      "localhost"
    ]
  },
  "routingx": {
    "strategy": "rules",
    "settings": {
      "domainStrategy": "IPIfNonMatch",
      "rules": [
        {
          "type": "field",
          "port": "1-52",
          "outboundTag": "direct"
        },
        {
          "type": "field",
          "port": "54-79",
          "outboundTag": "direct"
        },
        {
          "type": "field",
          "port": "81-442",
          "outboundTag": "direct"
        },
        {
          "type": "field",
          "port": "444-65535",
          "outboundTag": "direct"
        },
        {
          "type": "chinasites",
          "outboundTag": "direct"
        },
        {
          "type": "field",
          "ip": [
            "0.0.0.0/8",
            "10.0.0.0/8",
            "100.64.0.0/10",
            "127.0.0.0/8",
            "169.254.0.0/16",
            "172.16.0.0/12",
            "192.0.0.0/24",
            "192.0.2.0/24",
            "192.168.0.0/16",
            "198.18.0.0/15",
            "198.51.100.0/24",
            "203.0.113.0/24",
            "::1/128",
            "fc00::/7",
            "fe80::/10"
          ],
          "outboundTag": "direct"
        },
        {
          "type": "chinaip",
          "outboundTag": "direct"
        }
      ]
    }
  },
  "transport": {
    "kcpSettings": {
      "mtu": 1350,
      "tti": 20,
      "uplinkCapacity": 12,
      "downlinkCapacity": 12,
      "congestion": false,
      "readBufferSize": 1,
      "writeBufferSize": 1,
      "header": {
        "type": "none"
      }
    }
  }
}

```

- [Project V ](https://www.v2ray.com/chapter_00/install.html)
- [v2ray+BBR牛的一逼！](https://blog.verkey.org/209.html)
