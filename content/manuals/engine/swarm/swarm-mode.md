---
description: 在 Swarm 模式下运行 Docker Engine
keywords: 指南, Swarm 模式, 节点, Docker Engine
title: 在 Swarm 模式下运行 Docker Engine
---

当你首次安装并开始使用 Docker Engine 时，默认未启用 Swarm 模式。启用 Swarm 模式后，你将以“服务（service）”为核心进行工作，并通过 `docker service` 命令进行管理。

以 Swarm 模式运行 Engine 有两种方式：

* 新建一个 swarm（本文介绍）。
* [加入一个现有的 swarm](join-nodes.md)。

当你在本地机器上以 Swarm 模式运行 Engine 时，可以基于你构建的镜像或其他可用镜像来创建并测试服务。在生产环境中，Swarm 模式提供具备容错能力的集群管理平台，帮助你的服务持续运行并保持高可用。

本文假设你已在一台机器上安装了 Docker Engine，并将其作为 swarm 的管理节点（manager）。

如果尚未阅读，建议先了解[Swarm 模式关键概念](key-concepts.md)并试用[Swarm 模式教程](swarm-tutorial/_index.md)。

## 创建一个 swarm

当你运行创建 swarm 的命令时，Docker Engine 会启动并进入 Swarm 模式。

运行 [`docker swarm init`](/reference/cli/docker/swarm/init.md) 可在当前节点上创建一个单节点的 swarm。Engine 会按如下方式完成初始化：

* 将当前节点切换到 Swarm 模式。
* 创建一个名为 `default` 的 swarm。
* 将当前节点指定为该 swarm 的管理节点领导者（leader manager）。
* 使用机器的主机名为节点命名。
* 配置管理节点在一个活动的网络接口上监听 `2377` 端口。
* 将当前节点的可用性设置为 `Active`，表示它可以从调度器接收任务。
* 为参与该 swarm 的 Engine 启动一个内部的分布式数据存储，以维持整个 swarm 及其上所有服务的一致视图。
* 默认为该 swarm 生成自签名的根 CA。
* 默认生成用于 worker 与 manager 节点加入 swarm 的令牌（tokens）。
* 创建一个名为 `ingress` 的覆盖网络（overlay），用于将服务端口发布到 swarm 之外。
* 为你的网络创建默认的覆盖网络 IP 地址池与子网掩码。

`docker swarm init` 的输出中会提供在向 swarm 添加新的 worker 节点时所需的连接命令：

```console
$ docker swarm init
Swarm initialized: current node (dxn1zf6l61qsb1josjja83ngz) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

### 配置默认地址池

默认情况下，Swarm 模式为全局范围（overlay）的网络使用默认地址池 `10.0.0.0/8`。任何未显式指定子网的网络，都会从该地址池中按顺序分配一个子网。在某些情况下，你可能希望为网络使用不同的默认 IP 地址池。

例如，如果默认的 `10.0.0.0/8` 范围与你网络中已分配的地址空间冲突，则最好为网络使用另一个地址范围，而无需让 swarm 的使用者每次都通过 `--subnet` 指定子网。

要配置自定义默认地址池，必须在初始化 swarm 时通过 `--default-addr-pool` 命令行选项定义地址池。该选项使用 CIDR 表示法来定义子网掩码。要为 Swarm 创建自定义地址池，至少需要定义一个默认地址池，并可选地定义默认地址池的子网掩码长度。例如，对于 `10.0.0.0/27`，掩码长度设置为 `27`。

Docker 会从 `--default-addr-pool` 指定的地址范围分配子网。例如，命令行选项 `--default-addr-pool 10.10.0.0/16` 表示 Docker 将从该 `/16` 范围中分配子网。如果未指定 `--default-addr-pool-mask-len` 或显式设为 24，则会得到 256 个 `/24` 网络，形如 `10.10.X.0/24`。

子网范围来自 `--default-addr-pool`（例如 `10.10.0.0/16`）。其中的 `16` 表示在该默认地址池范围内可创建的网络规模。`--default-addr-pool` 选项可重复多次，每次提供额外的地址范围供 Docker 用于 overlay 子网。

命令格式如下：

```console
$ docker swarm init --default-addr-pool <IP range in CIDR> [--default-addr-pool <IP range in CIDR> --default-addr-pool-mask-length <CIDR value>]
```

为 `10.20.0.0` 网络创建一个 `/16`（B 类）默认地址池的命令如下：

```console
$ docker swarm init --default-addr-pool 10.20.0.0/16
```

同时为 `10.20.0.0` 与 `10.30.0.0` 网络创建 `/16`（B 类）默认地址池，且为每个网络创建 `/26` 子网掩码的命令如下：

```console
$ docker swarm init --default-addr-pool 10.20.0.0/16 --default-addr-pool 10.30.0.0/16 --default-addr-pool-mask-length 26
```

在上述示例中，执行 `docker network create -d overlay net1` 会分配到 `10.20.0.0/26` 作为 `net1` 的子网；
执行 `docker network create -d overlay net2` 会分配到 `10.20.0.64/26` 作为 `net2` 的子网。如此继续，直到所有子网被分配完。

更多信息请参见以下页面：
- [Swarm 网络](./networking.md)：了解默认地址池的使用方式
- `docker swarm init` 的[命令行参考](/reference/cli/docker/swarm/init.md)：了解 `--default-addr-pool` 标志的详细说明

### 配置对外通告地址（advertise address）

管理节点使用对外通告地址，使 swarm 中的其他节点可以访问 Swarmkit API 与 overlay 网络。其他节点必须能够通过该对外通告地址访问管理节点。

如果你没有指定对外通告地址，Docker 会检查系统是否只有一个 IP 地址：若是，则默认使用该 IP 地址及监听端口 `2377`；若系统有多个 IP 地址，则必须显式指定正确的 `--advertise-addr`，以启用管理节点之间的通信与 overlay 网络：

```console
$ docker swarm init --advertise-addr <MANAGER-IP>
```

当其他节点用于访问首个管理节点的地址与该管理节点自身看到的地址不一致时，也必须指定 `--advertise-addr`。例如，在跨区域的云环境中，主机既有区域内访问的内网地址，也有区域外访问的外网地址。在这种情况下，请使用 `--advertise-addr` 指定外网地址，以便该节点将此信息传播给随后连接的其他节点。

参见 `docker swarm init` 的[命令行参考](/reference/cli/docker/swarm/init.md)以了解更多关于对外通告地址的细节。

### 查看加入命令或轮换（更新）swarm 加入令牌

节点加入 swarm 需要一个密钥令牌。worker 节点的令牌与 manager 节点的令牌不同。节点仅在加入时使用该令牌。对已加入的节点来说，轮换加入令牌不会影响其在 swarm 中的成员身份。轮换可以确保旧令牌不会被新的节点用来加入 swarm。

要检索包含 worker 节点加入令牌的加入命令，运行：

```console
$ docker swarm join-token worker

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377

This node joined a swarm as a worker.
```

要查看 manager 节点的加入命令与令牌，运行：

```console
$ docker swarm join-token manager

To add a manager to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-59egwe8qangbzbqb3ryawxzk3jn97ifahlsrw01yar60pmkr90-bdjfnkcflhooyafetgjod97sz \
    192.168.99.100:2377
```

仅输出令牌时可使用 `--quiet`：

```console
$ docker swarm join-token --quiet worker

SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c
```

请谨慎保管加入令牌，因为它是加入 swarm 所必需的“秘密”。尤其不要将令牌提交到版本控制系统中，这会让任何能访问源代码的人都有能力向该 swarm 添加新节点。manager 令牌尤为敏感，因为它允许新管理节点加入并获得对整个 swarm 的控制权。

我们建议在以下情况下轮换加入令牌：

* 令牌被误提交到版本库、群聊，或被意外打印到日志中。
* 你怀疑某个节点已被入侵。
* 你希望确保不再有新节点加入该 swarm。

此外，最佳实践是为所有“秘密”（包括 swarm 加入令牌）制定定期轮换计划。建议至少每 6 个月轮换一次。

运行 `docker swarm join-token --rotate` 以作废旧令牌并生成新令牌。使用该命令时，需要指定对 `worker` 还是 `manager` 节点进行轮换：

```console
$ docker swarm join-token  --rotate worker

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-2kscvs0zuymrsc9t0ocyy1rdns9dhaodvpl639j2bqx55uptag-ebmn5u927reawo27s3azntd44 \
    192.168.99.100:2377
```

## 了解更多

* [向 swarm 加入节点](join-nodes.md)
* `swarm init` 的[命令行参考](/reference/cli/docker/swarm/init.md)
* [Swarm 模式教程](swarm-tutorial/_index.md)
