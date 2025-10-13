---
title: Docker Engine
weight: 10
description: 查看 Docker Engine 的完整概览，包括安装方式、存储、网络等内容
keywords: Engine
params:
  sidebar:
    group: Open source
grid:
- title: Install Docker Engine
  description: 了解如何在你的发行版上安装开源的 Docker Engine。
  icon: download
  link: /engine/install
- title: Storage
  description: 在 Docker 容器中使用持久化数据。
  icon: database
  link: /storage
- title: Networking
  description: 管理容器之间的网络连接。
  icon: network_node
  link: /network
- title: Container logs
  description: 了解如何查看与读取容器日志。
  icon: text_snippet
  link: /config/containers/logging/
- title: Prune
  description: 清理未使用的资源。
  icon: content_cut
  link: /config/pruning
- title: Configure the daemon
  description: 深入了解 Docker 守护进程的配置选项。
  icon: tune
  link: /config/daemon
- title: Rootless mode
  description: 以非 root 权限运行 Docker。
  icon: security
  link: /engine/security/rootless
- title: Deprecated features
  description: 了解 Docker Engine 中已弃用、应停止使用的功能。
  icon: folder_delete
  link: /engine/deprecated/
- title: Release notes
  description: 查看最新版本的发行说明。
  icon: note_add
  link: /engine/release-notes
aliases:
- /edge/
- /engine/ce-ee-node-activate/
- /engine/migration/
- /engine/misc/
- /linux/
---

Docker Engine 是一款开源的容器化技术，用于构建并容器化你的应用。Docker Engine 采用客户端-服务端架构，包括：

- 一个长期运行的守护进程（Server）
  [`dockerd`](/reference/cli/dockerd)。
- 一组 API，供程序与 Docker 守护进程通信与下发指令。
- 一个命令行（CLI）客户端
  [`docker`](/reference/cli/docker/)。

CLI 通过 [Docker API](/reference/api/engine/_index.md) 控制或与守护进程交互（脚本或直接命令）。许多 Docker 应用也复用这些 API 与 CLI。守护进程负责创建与管理 Docker 对象，如镜像、容器、网络与卷。

了解更多，请参阅《[Docker 架构](/get-started/docker-overview.md#docker-architecture)》。

{{< grid >}}

## 许可

在大型企业环境中（员工数超过 250 或年营收超过 1,000 万美元）通过 Docker Desktop 获取的 Docker Engine 用于商业用途时，需要[付费订阅](https://www.docker.com/pricing/)。
开源许可为 Apache License, Version 2.0，完整文本见 [LICENSE](https://github.com/moby/moby/blob/master/LICENSE)。
