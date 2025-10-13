---
description: 了解 Swarm 节点的工作方式
keywords: docker, container, cluster, swarm mode, node
title: 节点的工作原理
weight: 10
aliases:
- /engine/swarm/how-swarm-mode-works/
---

Swarm 模式允许你创建由一个或多个 Docker Engine 组成的集群（swarm）。一个 swarm 由一个或多个节点构成：这些节点是运行 Docker Engine 的物理机或虚拟机。

节点分为两类：[管理节点](#manager-nodes) 与 [工作节点](#worker-nodes)。

![Swarm mode cluster](/engine/swarm/images/swarm-diagram.webp)

如果尚未了解，建议先阅读《[Swarm 模式概览](../_index.md)》与《[关键概念](../key-concepts.md)》。

## 管理节点 {#manager-nodes}

管理节点负责集群管理任务：

* 维护集群状态
* 调度服务
* 提供 Swarm 模式的[HTTP API 端点](/reference/api/engine/_index.md)

管理节点基于 [Raft](https://raft.github.io/raft.pdf) 实现维护整个 swarm 及其上运行服务的一致内部状态。出于测试目的，可以只运行一个管理节点的集群；若该管理节点发生故障，你的服务仍会继续运行，但需要创建一个新集群来恢复管理能力。

要充分利用 Swarm 模式的容错能力，建议根据你的高可用需求，部署奇数个管理节点。当存在多个管理节点时，可以在不中断的情况下从单个管理节点故障中恢复。

* 3 管理节点的集群最多容忍 1 个管理节点失效。
* 5 管理节点的集群最多同时容忍 2 个管理节点失效。
* 集群中若有奇数个 `N` 管理节点，最多可容忍 `(N-1)/2` 个失效。
Docker 建议一个 swarm 的管理节点数量最多为 7。

    > [!IMPORTANT]
    >
    > 增加更多管理节点并不意味着更高的可扩展性或性能；通常恰恰相反。

## 工作节点 {#worker-nodes}

工作节点也是 Docker Engine 实例，其唯一职责是运行容器。工作节点不参与 Raft 分布式状态、不做调度决策，也不提供 Swarm 模式 HTTP API。

你可以创建仅包含一个管理节点的集群，但至少需要有一个管理节点才能拥有工作节点。默认情况下，所有管理节点同时也充当工作节点。在仅有一个管理节点的集群中，你仍可运行如 `docker service create` 等命令，调度器会把所有任务放置在本地引擎上。

在多节点集群中，如要阻止调度器将任务放置到管理节点，可将该管理节点的可用性设置为 `Drain`。调度器会优雅地停止处于 `Drain` 模式节点上的任务，并将这些任务调度到 `Active` 节点；同时不会再把新任务分配到 `Drain` 节点。

参见 [`docker node update`](/reference/cli/docker/node/update.md) 以了解如何更改节点可用性。

## 变更角色

你可以通过运行 `docker node promote` 将某个工作节点提升为管理节点。例如，当你计划下线一个管理节点进行维护时，可以提升另一个工作节点。参见 [node promote](/reference/cli/docker/node/promote.md)。

你也可以将管理节点降级为工作节点。参见 [node demote](/reference/cli/docker/node/demote.md)。


## 延伸阅读

* 了解 Swarm 模式下[服务](services.md)的工作方式。
* 了解 Swarm 模式下 [PKI](pki.md) 的工作方式。
