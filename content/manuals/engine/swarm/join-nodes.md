---
description: 向 swarm 添加工作节点与管理节点
keywords: 指南, Swarm 模式, 节点
title: 向 swarm 加入节点
---

当你首次创建一个 swarm 时，你会让单个 Docker Engine 进入 Swarm 模式。要充分发挥 Swarm 模式的能力，你可以向该 swarm 添加更多节点：

* 添加工作节点可提升容量。当你将服务部署到一个 swarm 时，引擎会把任务调度到可用节点上，无论这些节点是工作节点还是管理节点。向 swarm 添加更多工作节点，可以在不影响管理节点 Raft 共识的前提下提升任务处理规模。
* 管理节点提升容错能力。管理节点负责 swarm 的编排与集群管理。在多个管理节点中，会有一个领导者（leader）负责执行编排任务。若领导者宕机，其余管理节点会选举新的领导者并恢复对 swarm 状态的编排与维护。默认情况下，管理节点同样会运行任务。

Docker Engine 是否以及以何种角色加入 swarm，取决于你传给 `docker swarm join` 命令的加入令牌（join-token）。节点只在加入时使用该令牌。之后如果你轮换令牌，不会影响已在 swarm 中的节点。参见[在 Swarm 模式下运行 Docker Engine](swarm-mode.md#view-the-join-command-or-update-a-swarm-join-token)。

## 以工作节点身份加入

在某个管理节点上运行以下命令，获取包含工作节点加入令牌的加入命令：

```console
$ docker swarm join-token worker

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377
```

在工作节点上执行上述输出中的命令以加入 swarm：

```console
$ docker swarm join \
  --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
  192.168.99.100:2377

This node joined a swarm as a worker.
```

`docker swarm join` 命令会执行以下操作：

* 将当前节点上的 Docker Engine 切换到 Swarm 模式。
* 向管理节点请求 TLS 证书。
* 使用机器的主机名为节点命名。
* 根据提供的 swarm 令牌，使用管理节点的监听地址将当前节点加入 swarm。
* 将当前节点的可用性设置为 `Active`，表示它可以从调度器接收任务。
* 将 `ingress` 覆盖网络扩展到当前节点。

## 以管理节点身份加入

当你运行 `docker swarm join` 并传入管理节点的令牌时，Docker Engine 会像工作节点一样切换到 Swarm 模式。同时，管理节点会参与 Raft 共识。新加入的管理节点状态应当为 `Reachable`，而原有的管理节点仍为 swarm 的 `Leader`。

Docker 推荐每个集群使用三或五个管理节点以实现高可用。由于 Swarm 模式下的管理节点通过 Raft 共享数据，管理节点数量必须为奇数。只要超过半数的管理节点可用，swarm 就能继续对外提供服务。

关于管理节点与 swarm 运维的更多细节，参见[运维与维护由多个 Docker Engine 组成的 swarm](admin_guide.md)。

在某个管理节点上运行以下命令，获取包含管理节点加入令牌的加入命令：

```console
$ docker swarm join-token manager

To add a manager to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-61ztec5kyafptydic6jfc1i33t37flcl4nuipzcusor96k7kby-5vy9t8u35tuqm7vh67lrz9xp6 \
    192.168.99.100:2377
```

在新管理节点上执行输出中的命令，使其加入该 swarm：

```console
$ docker swarm join \
  --token SWMTKN-1-61ztec5kyafptydic6jfc1i33t37flcl4nuipzcusor96k7kby-5vy9t8u35tuqm7vh67lrz9xp6 \
  192.168.99.100:2377

This node joined a swarm as a manager.
```

## 了解更多

* `swarm join` 的[命令行参考](/reference/cli/docker/swarm/join.md)
* [Swarm 模式教程](swarm-tutorial/_index.md)
