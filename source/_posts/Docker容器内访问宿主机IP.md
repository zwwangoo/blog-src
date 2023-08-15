---
title: Docker容器内访问宿主机IP
date: 2023-08-15
tags: [Linux, Docker]

---



## 1、使用特殊DNS名称`host.docker.internal`

参考：[从容器连接到宿主机的服务](https://docs.docker.com/desktop/networking/#i-want-to-connect-from-a-container-to-a-service-on-the-host)

使用特殊的DNS名称`host.docker.internal`作为主机的内部IP地址，然后使用：

```bash
$ ping host.docker.internal
PING host.docker.internal (192.168.65.2) 56(84) bytes of data.
64 bytes from host.docker.internal (192.168.65.2): icmp_seq=1 ttl=37 time=3.03 ms
64 bytes from host.docker.internal (192.168.65.2): icmp_seq=2 ttl=37 time=2.42 ms
```

**注意该DNS 只支持 Mac 与 Windows 中 desktop 这种环境，并不支持在 Linux 中使用，所以不能直接使用。**


## 2、可以在运行时容器内添加一条解析`--add-host="host.docker.internal:host-gateway"`，建议Docker版本大于20，如果使用docker-compose.yaml则

```yaml
services:
  myservice:
    extra_hosts:
      - host.docker.internal:host-gateway
```

参考[https://github.com/docker/for-linux/issues/264](https://github.com/docker/for-linux/issues/264)

实测可能会报错：

```
Creating network "t_default" with the default driver
Creating t_myservice_1 ... error

ERROR: for t_myservice_1  Cannot create container for service myservice: invalid IP address in add-host: "host-gateway"

ERROR: for myservice  Cannot create container for service myservice: invalid IP address in add-host: "host-gateway"
ERROR: Encountered errors while bringing up the project.
```


## 3、设置容器网络，指定网关`172.15.234.1`作为宿主机访问ip

```yaml
services:
  myservice:
    environment:
	    - REDIS_URL=redis://172.15.234.1:6379/1
    networks:
      my_net:
        aliases:
          - myservice_net

networks:
  my_net:
    driver: bridge
    driver_opts:
      com.docker.network.enable_ipv6: "false"
    ipam:
      driver: default
      config:
      - subnet: 172.15.234.0/24
        gateway: 172.15.234.1
```