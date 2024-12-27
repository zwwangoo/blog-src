---

title: informix常用命令
date: 2024-09-13
tags: [ 数据库 ]

---

- 启动 informix 命令：　`oninit -vy`
- 关闭 informix 命令：　`onmode -ky`

1、查询集群信息
```
onstat -g cluster

# 或者执行sql
EXECUTE FUNCTION task("onstat", "-g cluster")
```

1.1、查看数据库信息

```
onstat -g dri
```