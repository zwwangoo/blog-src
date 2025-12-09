---
title: Docker 构建跨平台镜像：如何在 Mac (ARM) 上构建 Linux (AMD64) 镜像
date: 2025-12-09
tags:
  - 工具使用
---

## 场景 0：使用 Docker CLI (基础)

如果你只是临时构建一个简单的镜像，不使用编排工具，直接在 `docker build` 命令中加上 `--platform` 参数即可。

**命令示例：**

```bash
docker build --platform linux/amd64 -t my-image:latest .
```

## 场景 1：使用 Docker Compose (进阶)

在 `docker-compose build` 执行时，并不像 `docker build` 那样直接支持 `--platform` 参数。我们可以通过以下两种方式来实现。

### 方法 1：在 `docker-compose.yml` 文件中指定 (推荐)

这是最标准、最稳定的方法。通过在 `docker-compose.yml` 的服务定义中直接添加 `platform` 字段，将架构配置固化在代码中。

**适用场景：**

- 你的项目明确需要在特定架构（如 Linux/AMD64）服务器上运行。
- 团队成员混合使用 Intel 和 Apple Silicon 芯片，需要保证构建产物的一致性。

**配置示例：**

```yaml
version: '3.8'  # 注意：新版 Docker Compose (V2) 已忽略 version 字段，但保留也无妨

services:
  web:
    build: .
    # 核心配置：在这里强制指定平台
    platform: linux/amd64
    image: my-web-app
    ports:
      - "8080:80"
```

配置后，无论在什么机器上运行 `docker-compose build`，Docker 都会自动使用 `linux/amd64` 架构进行构建。
### 方法 2：使用环境变量 (临时指定)

如果你不想修改 `docker-compose.yml` 文件（例如不想把生产环境配置硬编码进开发文件），或者需要在不同架构间临时切换，可以使用 `DOCKER_DEFAULT_PLATFORM` 环境变量。

**适用场景：**

- CI/CD 流水线中动态指定构建架构。
- 在 M1/M2 Mac 上临时想要强制构建 AMD64 镜像进行测试，但不想改动代码。

**操作命令：**

你可以选择临时设置环境变量并执行命令：

```bash
# 方式 A：单行命令（推荐，只影响当前执行）
DOCKER_DEFAULT_PLATFORM=linux/amd64 docker-compose build

# 方式 B：先 export 再执行（影响当前终端会话）
export DOCKER_DEFAULT_PLATFORM=linux/amd64
docker-compose build
```

> **注意：** 使用此环境变量会强制将该 Compose 文件中的**所有**服务都尝试以指定平台构建。