---
description: Docker Engine Swarm 模式概览
keywords: docker, 容器, 集群, swarm, docker engine
title: Swarm 模式
weight: 80
aliases:
- /api/swarm-api/
- /engine/userguide/networking/overlay-standalone-swarm/
- /network/overlay-standalone.swarm/
- /release-notes/docker-swarm/
- /swarm/
- /swarm/api/
- /swarm/configure-tls/
- /swarm/discovery/
- /swarm/get-swarm/
- /swarm/install-manual/
- /swarm/install-w-machine/
- /swarm/multi-host-networking/
- /swarm/multi-manager-setup/
- /swarm/networking/
- /swarm/overview/
- /swarm/plan-for-production/
- /swarm/provision-with-machine/
- /swarm/reference/
- /swarm/reference/create/
- /swarm/reference/help/
- /swarm/reference/join/
- /swarm/reference/list/
- /swarm/reference/manage/
- /swarm/reference/swarm/
- /swarm/release-notes/
- /swarm/scheduler/
- /swarm/scheduler/filter/
- /swarm/scheduler/rescheduling/
- /swarm/scheduler/strategy/
- /swarm/secure-swarm-tls/
- /swarm/status-code-comparison-to-docker/
- /swarm/swarm-api/
- /swarm/swarm_at_scale/
- /swarm/swarm_at_scale/02-deploy-infra/
- /swarm/swarm_at_scale/03-create-cluster/
- /swarm/swarm_at_scale/04-deploy-app/
- /swarm/swarm_at_scale/about/
- /swarm/swarm_at_scale/deploy-app/
- /swarm/swarm_at_scale/deploy-infra/
- /swarm/swarm_at_scale/troubleshoot/
---

{{% include "swarm-mode.md" %}}

当前版本的 Docker 内置 Swarm 模式，用于原生管理由多个 Docker Engine 组成的集群（称为 swarm）。你可以使用 Docker CLI 创建一个 swarm、向 swarm 部署应用服务，并管理 swarm 的运行行为。

Docker 的 Swarm 模式内置于 Docker Engine。本页所述的 Swarm 模式不要与已停止维护的 [Docker Classic Swarm](https://github.com/docker/classicswarm) 混淆。

## 功能亮点

### 与 Docker Engine 集成的集群管理

使用 Docker Engine 的 CLI 来创建由多个 Docker Engine 组成的 swarm，并在其上部署应用服务。你无需额外的编排软件即可创建或管理一个 swarm。

### 去中心化设计

无需在部署时人为区分节点角色，Docker Engine 会在运行时处理相应的角色特化。你可以通过 Docker Engine 同时部署两类节点（manager 与 worker）。这意味着你可以从同一个磁盘镜像构建出完整的 swarm。

### 声明式服务模型

Docker Engine 采用声明式方式，让你定义应用中各个服务的“期望状态”。例如，你可以描述一个由 Web 前端、消息队列服务以及数据库后端组成的应用。

### 扩缩容

对于每个服务，你可以声明希望运行的任务（task）数量。当你进行扩容或缩容时，swarm 管理节点会通过增加或移除任务来自动调节，以保持期望状态。

### 期望状态收敛

swarm 管理节点会持续监控集群状态，并协调实际状态与期望状态之间的差异。比如，你让某个服务运行 10 个容器副本，如果承载其中 2 个副本的某个工作节点宕机，管理节点会创建 2 个新的副本来替换宕掉的副本，并把它们分配到当前可用的工作节点上。

### 多主机网络

你可以为服务指定一个 overlay 覆盖网络。swarm 管理节点会在初始化或更新应用时，自动为该覆盖网络中的容器分配地址。

### 服务发现

管理节点会为 swarm 中的每个服务分配唯一的 DNS 名称，并对正在运行的容器进行负载均衡。你可以通过内置于 swarm 的 DNS 服务器查询在 swarm 中运行的所有容器。

### 负载均衡

你可以将服务的端口暴露给外部负载均衡器。在 swarm 内部，你也可以指定如何在各节点之间分布服务容器。

### 默认安全

swarm 中的每个节点都会强制启用 TLS 双向认证与加密，以保护与其他所有节点之间的通信。你可以选择使用自签名根证书，或来自自定义根 CA 的证书。

### 滚动更新

在发布期间，你可以按批次将服务更新增量地应用到各节点。swarm 管理节点允许你控制服务在不同节点批次之间的部署延迟。如果出现问题，你可以回滚到先前版本的服务。

## 下一步

* 学习 Swarm 模式的[关键概念](key-concepts.md)。
* 通过[Swarm 模式教程](swarm-tutorial/_index.md)快速上手。
* 探索 Swarm 模式相关 CLI 命令：
  * [swarm init](/reference/cli/docker/swarm/init.md)
  * [swarm join](/reference/cli/docker/swarm/join.md)
  * [service create](/reference/cli/docker/service/create.md)
  * [service inspect](/reference/cli/docker/service/inspect.md)
  * [service ls](/reference/cli/docker/service/ls.md)
  * [service rm](/reference/cli/docker/service/rm.md)
  * [service scale](/reference/cli/docker/service/scale.md)
  * [service ps](/reference/cli/docker/service/ps.md)
  * [service update](/reference/cli/docker/service/update.md)
