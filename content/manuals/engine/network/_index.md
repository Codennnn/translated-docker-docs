---
title: 网络概览
linkTitle: 网络
weight: 30
description: 从容器视角了解网络如何工作
keywords: networking, container, standalone, IP address, DNS resolution
aliases:
- /articles/networking/
- /config/containers/container-networking/
- /engine/tutorials/networkingcontainers/
- /engine/userguide/networking/
- /engine/userguide/networking/configure-dns/
- /engine/userguide/networking/default_network/binding/
- /engine/userguide/networking/default_network/configure-dns/
- /engine/userguide/networking/default_network/container-communication/
- /engine/userguide/networking/dockernetworks/
- /network/
---

容器网络指的是容器之间，或容器与非 Docker 工作负载之间，进行连接与通信的能力。

容器默认启用网络，并且可以发起出站连接。容器并不了解它所连接的是哪种网络，也不知道它的对端是否同样是 Docker 工作负载。从容器的视角，它只看到一个带有 IP 地址、网关、路由表、DNS 服务等细节的网络接口——除非该容器使用 `none` 网络驱动。

本文从容器的视角介绍网络及其相关概念。
本文不涉及不同操作系统下 Docker 网络的实现细节。关于 Docker 在 Linux 上如何修改 `iptables` 规则，请参见[数据包过滤与防火墙](packet-filtering-firewalls.md)。

## 用户自定义网络

你可以创建自定义的用户网络，并将多个容器连接到同一个网络。连接到用户自定义网络后，容器可以通过容器的 IP 地址或容器名称相互通信。

下面的示例使用 `bridge` 网络驱动创建一个网络，并在该网络中运行一个容器：

```console
$ docker network create -d bridge my-net
$ docker run --network=my-net -itd --name=container3 busybox
```

### 驱动

以下网络驱动默认可用，并提供核心网络功能：

| 驱动      | 说明                                                     |
| :-------- | :------------------------------------------------------- |
| `bridge`  | 默认网络驱动。                                           |
| `host`    | 移除容器与 Docker 主机之间的网络隔离。                   |
| `none`    | 将容器与主机及其他容器完全隔离。                         |
| `overlay` | 用于连接多个 Docker 守护进程的 Overlay 网络。            |
| `ipvlan`  | 对 IPv4 与 IPv6 地址分配提供完全控制。                   |
| `macvlan` | 为容器分配一个 MAC 地址。                                |

关于不同驱动的更多信息，参见[网络驱动概览](./drivers/_index.md)。

### 连接到多个网络

容器可以连接到多个网络。

例如，一个前端容器可能连接到具有外部访问能力的 `bridge` 网络，同时还连接到一个
[`--internal`](/reference/cli/docker/network/create/#internal) 网络，以便与无需外网访问的后端服务容器通信。

容器也可以连接不同类型的网络。例如，使用 `ipvlan` 网络提供互联网访问，同时连接 `bridge` 网络以访问本地服务。

当发送数据包时，如果目标地址位于某个直接连接的网络中，数据包会直接发往该网络；否则会发往默认网关，由其路由到目的地。在上述示例中，`ipvlan` 网络的网关必须是默认网关。

默认网关由 Docker 选择，并且当容器的网络连接发生变化时，这一选择可能会改变。
如果希望在创建容器或连接新网络时让 Docker 选择特定默认网关，可以设置网关优先级。参见 [`docker run`](/reference/cli/docker/container/run.md) 与
[`docker network connect`](/reference/cli/docker/network/connect.md) 命令中的 `gw-priority` 选项。

`gw-priority` 的默认值为 `0`。优先级最高的网络中的网关会被选为默认网关。因此，当希望某个网络始终作为默认网关时，将其 `gw-priority` 设为 `1` 即可。

```console
$ docker run --network name=gwnet,gw-priority=1 --network anet1 --name myctr myimage
$ docker network connect anet2 myctr
```

## 容器网络（container: 模式）

除了用户自定义网络外，你还可以使用 `--network container:<name|id>` 的形式，直接将一个容器附加到另一个容器的网络栈。

在 `container:` 网络模式下，以下标志不受支持：

- `--add-host`
- `--hostname`
- `--dns`
- `--dns-search`
- `--dns-option`
- `--mac-address`
- `--publish`
- `--publish-all`
- `--expose`

下面的示例先运行一个 Redis 容器，并让 Redis 仅绑定到 `localhost`；随后通过 `redis-cli` 在 `localhost` 接口上连接该 Redis 实例。

```console
$ docker run -d --name redis example/redis --bind 127.0.0.1
$ docker run --rm -it --network container:redis example/redis-cli -h 127.0.0.1
```

## 端口发布

默认情况下，当你使用 `docker create` 或 `docker run` 创建或运行容器时，处于桥接网络的容器不会向外部世界暴露任何端口。
使用 `--publish` 或 `-p` 标志可以让容器端口对桥接网络之外的服务可见。
这会在宿主机上创建一条防火墙规则，将容器端口映射到 Docker 主机上的端口，使之对外可达。
示例如下：

| 参数值                          | 说明                                                                                                                             |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `-p 8080:80`                    | 将 Docker 主机的 `8080` 端口映射到容器内的 TCP `80` 端口。                                                                        |
| `-p 192.168.1.100:8080:80`      | 将 Docker 主机 IP `192.168.1.100` 的 `8080` 端口映射到容器内的 TCP `80` 端口。                                                    |
| `-p 8080:80/udp`                | 将 Docker 主机的 `8080` 端口映射到容器内的 UDP `80` 端口。                                                                        |
| `-p 8080:80/tcp -p 8080:80/udp` | 将主机的 TCP `8080` 端口映射到容器内 TCP `80`，同时将主机的 UDP `8080` 端口映射到容器内 UDP `80`。 |

> [!IMPORTANT]
>
> 默认情况下，发布容器端口是不安全的。这意味着，一旦发布容器端口，它不仅对 Docker 主机可见，也会对外部网络可见。
>
> 如果在发布规则中包含本地回环地址（`127.0.0.1` 或 `::1`），则只有 Docker 主机及其上的容器可以访问该已发布端口。
>
> ```console
> $ docker run -p 127.0.0.1:8080:80 -p '[::1]:8080:80' nginx
> ```
>
> > [!WARNING]
> >
> > 在 28.0.0 之前的版本中，同一二层网络（例如连接到同一交换机的主机）内的主机，可能仍可访问发布到本地回环地址的端口。
> > 了解更多信息，请参见
> > [moby/moby#45610](https://github.com/moby/moby/issues/45610)

如果只是希望其他容器可以访问该容器，则无需发布容器端口。
将这些容器连接到同一个网络（通常是[bridge 网络](./drivers/bridge.md)）即可实现容器间通信。

当端口映射未指定主机 IP、桥接网络仅支持 IPv4，且 `--userland-proxy=true`（默认）时，主机 IPv6 地址上的端口会映射到容器的 IPv4 地址。

关于端口映射的更多信息（包括如何禁用它并改用直达容器的路由），请参见[数据包过滤与防火墙](./packet-filtering-firewalls.md)。

## IP 地址与主机名

创建网络时默认启用 IPv4 地址分配，你可以通过 `--ipv4=false` 将其禁用；可以通过 `--ipv6` 启用 IPv6 地址分配。

```console
$ docker network create --ipv6 --ipv4=false v6net
```

默认情况下，容器会从其加入的每个 Docker 网络中各获得一个 IP 地址。
容器从该网络的 IP 子网中获取地址。Docker 守护进程会为容器执行动态子网划分与 IP 地址分配。
每个网络还具有默认的子网掩码与网关。

你可以让一个正在运行的容器连接到多个网络：要么在创建容器时多次传入 `--network` 标志，要么对已运行的容器使用 `docker network connect` 命令。
无论哪种方式，都可以通过 `--ip` 或 `--ip6` 指定容器在特定网络上的 IP 地址。

同样地，容器的主机名默认是其在 Docker 中的容器 ID。
你可以通过 `--hostname` 进行覆盖。
当使用 `docker network connect` 连接到已有网络时，可以通过 `--alias` 为容器在该网络上设置额外的网络别名。

## DNS 服务

容器默认使用与主机相同的 DNS 服务器，但你可以通过 `--dns` 覆盖该行为。

默认情况下，容器会继承 `/etc/resolv.conf` 中定义的 DNS 设置。
连接到默认 `bridge` 网络的容器会收到该文件的副本。
连接到[自定义网络](tutorials/standalone.md#use-user-defined-bridge-networks)的容器则使用 Docker 的内置 DNS 服务器。
内置 DNS 会将外部 DNS 查询转发给主机上配置的 DNS 服务器。

你可以在每个容器维度配置 DNS 解析：在启动容器时为 `docker run` 或 `docker create` 命令添加相应标志。
下表描述了与 DNS 配置相关的 `docker run` 可用标志：

| 参数           | 说明                                                                                                                                                                                                                                                   |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `--dns`        | 指定 DNS 服务器的 IP 地址。要指定多个 DNS 服务器，可多次使用 `--dns`。DNS 请求将从容器的网络命名空间发出，例如 `--dns=127.0.0.1` 指向的是容器自身的回环地址。                                                       |
| `--dns-search` | 指定用于解析非完全限定主机名的 DNS 搜索域。要指定多个搜索前缀，可多次使用 `--dns-search`。                                                                                                                         |
| `--dns-opt`    | 以键值对形式指定 DNS 选项及其值。有效选项请参考所用操作系统关于 `resolv.conf` 的文档。                                                                                                                             |
| `--hostname`   | 指定容器自用的主机名；若未指定，默认使用容器 ID。                                                                                                                                                                                                       |

### 自定义 hosts

容器内的 `/etc/hosts` 文件包含容器自身的主机名，以及 `localhost` 等常见条目。主机上的 `/etc/hosts` 中定义的自定义 hosts 不会被容器继承。
如需向容器传入额外的主机名解析条目，请参见 `docker run` 参考文档的[向容器 hosts 文件添加条目](/reference/cli/docker/container/run.md#add-host)。

## 代理服务器

如果你的容器需要使用代理服务器，请参见[使用代理服务器](/manuals/engine/daemon/proxy.md)。
