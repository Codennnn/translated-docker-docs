---
title: Docker Build 概览
weight: 10
description: 了解 Docker Build 及其组成部分。
keywords: build, buildkit, buildx, architecture
aliases:
- /build/install-buildx/
- /build/architecture/
---

Docker Build 采用客户端-服务器架构，其中：

- Client：Buildx 是用于运行与管理构建的客户端和用户界面。
- Server：BuildKit 是负责执行构建的服务器（构建器）。

当你发起构建时，Buildx 客户端会向 BuildKit 后端发送构建请求。BuildKit 解析构建指令并执行各个步骤。构建产物要么返回给客户端，要么上传到仓库（例如 Docker Hub）。

在 Docker Desktop 与 Docker Engine 中，Buildx 与 BuildKit 开箱即用。执行 `docker build` 命令时，你实际上是通过 Buildx 使用 Docker 捆绑的默认 BuildKit 来运行构建。

## Buildx

Buildx 是用于运行构建的 CLI 工具。`docker build` 命令是对 Buildx 的封装。当你调用 `docker build` 时，Buildx 会解析构建选项并向 BuildKit 后端发送构建请求。

Buildx 的能力不仅限于运行构建。你还可以使用 Buildx 创建和管理 BuildKit 后端（称为 builder）。它也支持管理仓库中的镜像，并发运行多个构建等功能。

Docker Buildx 在 Docker Desktop 中默认安装。你也可以从源码构建该 CLI 插件，或从 GitHub 仓库下载二进制并手动安装。更多信息请参见 GitHub 上的 [Buildx README](https://github.com/docker/buildx#manual-download)。

> [!NOTE]
> 虽然 `docker build` 在底层调用了 Buildx，但该命令与标准的 `docker buildx build` 仍存在一些细微差异。详情参见 [Difference between `docker build` and `docker buildx build`](../builders/_index.md#difference-between-docker-build-and-docker-buildx-build)。

## BuildKit

BuildKit 是负责执行构建任务的守护进程。

一次构建从调用 `docker build` 命令开始。Buildx 解析你的构建命令，并向 BuildKit 后端发送构建请求。构建请求包括：

- Dockerfile
- 构建参数
- 导出选项
- 缓存选项

BuildKit 解析构建指令并执行构建步骤。在构建执行期间，Buildx 会监控构建状态并将进度输出到终端。

如果构建需要来自客户端的资源（例如本地文件或构建机密），BuildKit 会向 Buildx 请求所需资源。

这也是 BuildKit 相比早期 Docker 所用旧版构建器更高效的原因之一：BuildKit 仅在需要时请求构建所需的资源；而旧版构建器通常会复制整个本地文件系统。

BuildKit 可能向 Buildx 请求的资源示例包括：

- 本地文件系统构建上下文
- 构建机密
- SSH 套接字
- 仓库认证令牌

更多关于 BuildKit 的信息，参见 [BuildKit](/manuals/build/buildkit/_index.md)。
