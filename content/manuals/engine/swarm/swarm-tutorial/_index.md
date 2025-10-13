---
description: Docker Engine Swarm 模式入门教程
keywords: tutorial, cluster management, swarm mode, docker engine, get started
title: Swarm 模式入门
toc_max: 4
---

本教程将带你了解 Docker Engine 的 Swarm 模式功能。开始之前，建议先阅读《[关键概念](../key-concepts.md)》。

本教程将引导你完成以下内容：

* 以 Swarm 模式初始化 Docker Engine 集群
* 向 swarm 添加节点
* 将应用服务部署到 swarm
* 当一切运行起来后，如何管理 swarm

本教程通过在终端中执行 Docker Engine 的 CLI 命令完成操作。

如果你刚接触 Docker，请先阅读《[关于 Docker Engine](../../_index.md)》。

## 准备工作

要完成本教程，你需要：

* [三台可通过网络互联、且已安装 Docker 的 Linux 主机](#three-networked-host-machines)
* [管理节点的 IP 地址](#the-ip-address-of-the-manager-machine)
* [在主机之间开放必要的端口](#open-protocols-and-ports-between-the-hosts)

### 三台可互联的主机 {#three-networked-host-machines}

本教程需要三台已安装 Docker 且能通过网络互通的 Linux 主机。它们可以是物理机、虚拟机、Amazon EC2 实例，或其他托管方式。参见《[部署到 Swarm](/guides/swarm-deploy.md#prerequisites)》以获取一套可行的主机准备方案。

其中一台作为管理节点（记为 `manager1`），另外两台作为工作节点（`worker1` 与 `worker2`）。

> [!NOTE]
>
> 你也可以在单节点上测试本教程中的许多步骤，此时只需一台主机。多节点相关的命令将无法使用，但你仍可以初始化一个 swarm、创建服务并进行扩缩容。

#### 在 Linux 机器上安装 Docker Engine

如果使用 Linux 物理机或云主机作为宿主，请按照你的平台对应的《[Linux 安装说明](../../install/_index.md)》进行安装。启动这三台机器后即可开始。你可以在 Linux 上同时验证单节点与多节点的 Swarm 场景。

### 管理节点的 IP 地址 {#the-ip-address-of-the-manager-machine}

该 IP 必须分配给宿主机操作系统可用的某个网络接口。swarm 中的所有节点都需要通过该 IP 地址连接到管理节点。

由于其他节点会通过管理节点的 IP 地址与其通信，建议你使用固定 IP。

你可以在 Linux 或 macOS 上运行 `ifconfig` 查看可用的网络接口列表。

本文示例使用 `manager1` ：`192.168.99.100`。

### 主机间需开放的协议与端口 {#open-protocols-and-ports-between-the-hosts}

以下端口必须可用。在某些系统上，这些端口默认已开放：

* 端口 `2377`（TCP）：用于与管理节点通信（包括管理节点之间）
* 端口 `7946`（TCP/UDP）：用于 overlay 网络的节点发现
* 端口 `4789`（UDP，可配置）：用于 overlay 网络的数据流量

如果你计划创建启用加密的 overlay 网络（`--opt encrypted`），还需要确保允许 IP 协议 50（IPSec ESP）流量通过。

端口 `4789` 是 Swarm 数据路径端口（也称 VXLAN 端口）的默认值。务必阻止任何不受信任的流量到达该端口，因为 VXLAN 本身不提供认证。该端口只应对可信网络开放，绝不要在边界防火墙上暴露。

如果承载 Swarm 流量的网络并非完全可信，强烈建议使用加密的 overlay 网络。若仅使用加密 overlay 网络，建议再进行如下加固：

* [自定义默认 ingress 网络](../networking.md) 以启用加密
* 仅在数据路径端口上接受加密数据包：

```bash
# 示例 iptables 规则（顺序及其他工具可能需要按环境调整）
iptables -I INPUT -m udp --dport 4789 -m policy --dir in --pol none -j DROP
```

## 下一步

接下来你将创建一个 swarm。

{{< button text="创建一个 swarm" url="create-swarm.md" >}}
