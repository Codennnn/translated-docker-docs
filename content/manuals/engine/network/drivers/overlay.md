---
title: Overlay 网络驱动
description: 使用 overlay 网络的完整指南
keywords: network, overlay, user-defined, swarm, service
aliases:
- /config/containers/overlay/
- /engine/userguide/networking/overlay-security-model/
- /network/overlay/
- /network/drivers/overlay/
---

`overlay` 网络驱动可在多台 Docker 守护进程主机之间创建一个分布式网络。该网络“覆盖”在各主机的本地网络之上；当启用加密时，连接其上的容器可以安全通信。Docker 会透明地将每个数据包路由到正确的主机与目标容器。

你可以像创建用户自定义 `bridge` 网络那样，使用 `docker network create` 创建用户自定义的 `overlay` 网络。服务或容器可以同时连接到多个网络；它们只能在各自连接到的网络之间通信。

Overlay 网络常用于连接 Swarm 服务；也可用于连接运行在不同主机上的独立容器。对于独立容器场景，仍需启用 Swarm 模式以在主机间建立连接。

本文介绍 overlay 网络的通用概念及其在独立容器中的用法。关于 Swarm 服务中的 overlay 网络，请参见[管理 Swarm 服务网络](/manuals/engine/swarm/networking.md)。

## 创建 overlay 网络

开始之前，请确保参与的各主机之间可以互相通信。下表列出了参与 overlay 网络的每台主机需要放通的端口：

| 端口                   | 说明                                                                                                                                                                   |
| :--------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `2377/tcp`             | 默认 Swarm 控制面端口，可通过 [`docker swarm join --listen-addr`](/reference/cli/docker/swarm/join.md#--listen-addr-value) 配置                        |
| `4789/udp`             | 默认 overlay 流量端口，可通过 [`docker swarm init --data-path-addr`](/reference/cli/docker/swarm/init.md#data-path-port) 配置                         |
| `7946/tcp`, `7946/udp` | 节点间通信端口，固定不可配置                                                                                                                                           |

要创建一个可供其他 Docker 主机上的容器连接的 overlay 网络，运行以下命令：

```console
$ docker network create -d overlay --attachable my-attachable-overlay
```

`--attachable` 选项允许独立容器与 Swarm 服务同时连接到该 overlay 网络；若不加该选项，则只有 Swarm 服务可以连接。

你可以指定 IP 地址范围、子网、网关等选项。详见 `docker network create --help`。

## 在 overlay 网络上加密传输

使用 `--opt encrypted` 标志，可以对 overlay 网络上的应用数据进行加密传输：

```console
$ docker network create \
  --opt encrypted \
  --driver overlay \
  --attachable \
  my-attachable-multi-host-network
```

这会在 VXLAN 层启用 IPsec 加密。加密会带来一定的性能开销，建议在生产使用前进行充分测试。

> [!WARNING]
>
> 请不要将 Windows 容器连接到启用了加密的 overlay 网络。
>
> Windows 不支持 overlay 网络加密。Swarm 在 Windows 主机尝试连接加密的 overlay 网络时不会报错，但会产生以下影响：
>
> - Windows 容器无法与网络中的 Linux 容器通信
> - Windows 容器之间的数据流量不会被加密

## 将容器连接到 overlay 网络

将容器加入 overlay 网络后，它们无需在各自 Docker 主机上额外配置路由，即可相互通信。前提是这些主机必须加入同一个 Swarm。

让 `busybox` 容器加入名为 `multi-host-network` 的 overlay 网络：

```console
$ docker run --network multi-host-network busybox sh
```

> [!NOTE]
>
> 仅当 overlay 网络创建时带有 `--attachable` 标志，才支持独立容器加入。

## 容器发现

在 overlay 网络上发布容器端口，会将该端口对同一网络中的其他容器开放。容器可通过其名称进行 DNS 查询与发现。

| 参数值                          | 说明                                                                                                                                       |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `-p 8080:80`                    | 将容器内 TCP 80 端口映射为 overlay 网络上的 `8080` 端口。                                                                                 |
| `-p 8080:80/udp`                | 将容器内 UDP 80 端口映射为 overlay 网络上的 `8080` 端口。                                                                                 |
| `-p 8080:80/sctp`               | 将容器内 SCTP 80 端口映射为 overlay 网络上的 `8080` 端口。                                                                                |
| `-p 8080:80/tcp -p 8080:80/udp` | 将容器内 TCP 80 端口映射为 overlay 网络上的 TCP `8080`，并将容器内 UDP 80 端口映射为 overlay 网络上的 UDP `8080`。 |

## overlay 网络的连接数限制

受 Linux 内核限制，当同一主机上并置的容器数量达到 1000 个时，overlay 网络可能变得不稳定，容器间通信可能中断。

更多信息参见 [moby/moby#44973](https://github.com/moby/moby/issues/44973#issuecomment-1543747718)。

## 进一步阅读

- 学习[overlay 网络教程](/manuals/engine/network/tutorials/overlay.md)
- 了解[从容器视角的网络](../_index.md)
- 了解[独立 bridge 网络](bridge.md)
- 了解[Macvlan 网络](macvlan.md)
