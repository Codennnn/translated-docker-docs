---
title: Bridge 网络驱动
description: 关于使用用户自定义 bridge 网络与默认 bridge 的全部内容
keywords: network, bridge, user-defined, standalone
aliases:
- /config/containers/bridges/
- /engine/userguide/networking/default_network/build-bridges/
- /engine/userguide/networking/default_network/custom-docker0/
- /engine/userguide/networking/work-with-networks/
- /network/bridge/
- /network/drivers/bridge/
---

从网络角度看，bridge 网络是一种链路层设备，用于在不同网络段之间转发流量。bridge 可以是硬件设备，也可以是运行在主机内核中的软件设备。

在 Docker 中，bridge 网络基于软件网桥实现：连接到同一 bridge 网络的容器可以相互通信，同时与未连接到该网络的容器保持隔离。Docker 的 bridge 驱动会在主机上自动安装相关规则，使处于不同 bridge 网络的容器无法直接互通。

bridge 网络适用于运行在同一 Docker 守护进程主机上的容器。如果需要不同主机上的容器相互通信，可以在操作系统层面配置路由，或使用[overlay 网络](overlay.md)。

当 Docker 启动时，会自动创建一个[默认 bridge 网络](#use-the-default-bridge-network)（名称为 `bridge`）。除非另行指定，新启动的容器会连接到该网络。你也可以创建用户自定义的 bridge 网络。**用户自定义 bridge 网络优于默认 `bridge` 网络。**

## 用户自定义 bridge 与默认 bridge 的差异

- **用户自定义 bridge 提供容器间的自动 DNS 解析**。

  处于默认 bridge 网络的容器只能通过 IP 地址互相访问，除非使用已被视为遗留的 [`--link` 选项](../links.md)。在用户自定义 bridge 网络上，容器可以通过名称或别名相互解析。

  假设有一个包含 Web 前端与数据库后端的应用。如果将容器命名为 `web` 与 `db`，则无论应用栈运行在哪台 Docker 主机上，`web` 容器都可以通过主机名 `db` 连接到数据库容器。

  如果在默认 bridge 网络上运行同样的应用栈，你需要手动在容器之间创建链接（使用遗留的 `--link` 标志）。这些链接需要双向创建；当需要通信的容器超过两个时，复杂度会迅速上升。你也可以在容器内修改 `/etc/hosts` 文件，但这会引入难以调试的问题。

- **用户自定义 bridge 提供更好的隔离性**。

  未显式指定 `--network` 的所有容器都会被连接到默认 bridge 网络。这可能带来风险：彼此无关的栈/服务/容器因此能够通信。

  使用用户自定义网络可以提供一个作用域明确的网络，只有连接到该网络的容器才可通信。

- **可以在容器运行期间动态加入/移除用户自定义网络**。

  在容器生命周期内，你可以随时将其连接到或断开自定义网络。相对地，要将容器从默认 bridge 网络移除，则需要先停止容器并以不同的网络选项重新创建。

- **每个用户自定义网络都会创建一个可配置的 bridge**。

  如果容器使用默认 bridge 网络，你可以对其配置，但所有容器将共享相同设置（例如 MTU 与 `iptables` 规则）。此外，默认 bridge 的配置需要在 Docker 之外进行，并且需要重启 Docker。

  用户自定义 bridge 网络通过 `docker network create` 创建并配置。若不同的应用组有不同的网络需求，你可以在创建时分别进行配置。

- **默认 bridge 网络上通过 link 的容器会共享环境变量**。

  早期在两个容器间共享环境变量的唯一方式是使用 [`--link` 标志](../links.md) 将它们链接起来。用户自定义网络不支持这种变量共享，但有更好的替代方案，例如：

  - 多个容器通过 Docker 卷挂载包含共享信息的文件或目录。

  - 使用 `docker-compose` 同时启动多个容器，并在 Compose 文件中定义共享变量。

  - 使用 Swarm 服务替代独立容器，利用共享的[密钥](/manuals/engine/swarm/secrets.md)与[配置](/manuals/engine/swarm/configs.md)。

连接到同一用户自定义 bridge 网络的容器，彼此之间等同于所有端口可见。若希望不同网络中的容器或非 Docker 主机访问某个端口，必须使用 `-p` 或 `--publish` 将该端口“发布”。

## 选项

下表列出了使用 `bridge` 驱动创建自定义网络时，可通过 `--opt` 传入的驱动特定选项：

| 选项                                                                                             | 默认值                      | 说明                                                                                                 |
|-------------------------------------------------------------------------------------------------|-----------------------------|------------------------------------------------------------------------------------------------------|
| `com.docker.network.bridge.name`                                                                |                             | 创建 Linux bridge 时使用的接口名称。                                                                  |
| `com.docker.network.bridge.enable_ip_masquerade`                                                | `true`                      | 启用 IP 伪装（masquerading）。                                                                       |
| `com.docker.network.bridge.gateway_mode_ipv4`<br/>`com.docker.network.bridge.gateway_mode_ipv6` | `nat`                       | 控制外部连通性。见[数据包过滤与防火墙](packet-filtering-firewalls.md)。                               |
| `com.docker.network.bridge.enable_icc`                                                          | `true`                      | 启用或禁用容器间互联（Inter-Container Connectivity）。                                               |
| `com.docker.network.bridge.host_binding_ipv4`                                                   | 所有 IPv4 与 IPv6 地址      | 绑定容器端口时使用的默认 IP。                                                                         |
| `com.docker.network.driver.mtu`                                                                 | `0`（不限制）               | 设置容器网络的最大传输单元（MTU）。                                                                    |
| `com.docker.network.container_iface_prefix`                                                     | `eth`                       | 为容器网络接口设置自定义前缀。                                                                       |
| `com.docker.network.bridge.inhibit_ipv4`                                                        | `false`                     | 阻止 Docker 为该 bridge [分配 IPv4 地址](#skip-bridge-ip-address-configuration)。                    |

上述部分选项也可作为 `dockerd` CLI 的标志使用，你可以在启动 Docker 守护进程时用来配置默认的 `docker0` bridge。下表列出了与 `dockerd` CLI 标志的对应关系：

| 选项                                             | 标志        |
| ------------------------------------------------ | ----------- |
| `com.docker.network.bridge.name`                 | -           |
| `com.docker.network.bridge.enable_ip_masquerade` | `--ip-masq` |
| `com.docker.network.bridge.enable_icc`           | `--icc`     |
| `com.docker.network.bridge.host_binding_ipv4`    | `--ip`      |
| `com.docker.network.driver.mtu`                  | `--mtu`     |
| `com.docker.network.container_iface_prefix`      | -           |

Docker 守护进程支持 `--bridge` 标志，你可以用它自定义 `docker0` bridge。若需要在同一主机上运行多个守护进程实例，可使用此选项。详情参见[运行多个守护进程](/reference/cli/dockerd.md#run-multiple-daemons)。

### 默认主机绑定地址

当在端口发布选项（如 `-p 80` 或 `-p 8080:80`）中未指定主机地址时，默认会在主机的所有地址（IPv4 与 IPv6）上开放容器的 80 端口。

可以使用 bridge 网络驱动选项 `com.docker.network.bridge.host_binding_ipv4` 修改已发布端口的默认绑定地址。

尽管该选项名称包含 IPv4，你仍可以指定 IPv6 地址。

当默认绑定地址是某个特定接口上的地址时，容器端口将只能通过该地址访问。

将默认绑定地址设置为 `::` 表示仅在主机的 IPv6 地址上开放已发布端口；设置为 `0.0.0.0` 则表示在主机的 IPv4 与 IPv6 地址上均可用。

若希望仅限 IPv4，可在容器的发布选项中显式指定地址，例如：`-p 0.0.0.0:8080:80`。

## 管理用户自定义 bridge

使用 `docker network create` 命令创建用户自定义 bridge 网络。

```console
$ docker network create my-net
```

你可以指定子网、IP 地址范围、网关及其他选项。详情参阅
[docker network create](/reference/cli/docker/network/create.md#specify-advanced-options)
参考文档，或查看 `docker network create --help` 的输出。

使用 `docker network rm` 命令删除用户自定义 bridge 网络。若有容器仍连接于该网络，请先[断开连接](#disconnect-a-container-from-a-user-defined-bridge)。

```console
$ docker network rm my-net
```

> **背后发生了什么？**
>
> 当你创建/删除用户自定义 bridge，或将容器连接/断开该网络时，Docker 会调用与操作系统相关的工具管理底层网络设施（如添加/删除 bridge 设备，或在 Linux 上配置 `iptables` 规则）。这些都属于实现细节，交由 Docker 处理即可。

## 将容器连接到用户自定义 bridge

创建新容器时，你可以指定一个或多个 `--network` 标志。下面的示例将一个 Nginx 容器连接到 `my-net` 网络，并把容器的 80 端口发布到主机的 8080 端口，方便外部客户端访问。任何连接到 `my-net` 的其他容器都可以访问 `my-nginx` 容器上的所有端口，反之亦然。

```console
$ docker create --name my-nginx \
  --network my-net \
  --publish 8080:80 \
  nginx:latest
```

要将一个“正在运行”的容器连接到现有的用户自定义 bridge，请使用 `docker network connect`。如下命令把已在运行的 `my-nginx` 容器连接到已存在的 `my-net` 网络：

```console
$ docker network connect my-net my-nginx
```

## 将容器从用户自定义 bridge 断开

要将运行中的容器从用户自定义 bridge 断开，请使用 `docker network disconnect` 命令。如下命令将 `my-nginx` 从 `my-net` 网络断开：

```console
$ docker network disconnect my-net my-nginx
```

## 在用户自定义 bridge 网络中使用 IPv6

创建网络时，可以通过 `--ipv6` 标志启用 IPv6：

```console
$ docker network create --ipv6 --subnet 2001:db8:1234::/64 my-net
```

如果未提供 `--subnet` 选项，将自动选择一个唯一本地地址（ULA）前缀。

## 仅 IPv6 的 bridge 网络

若要跳过在 bridge 及其容器上的 IPv4 地址配置，可在创建网络时使用 `--ipv4=false`，并通过 `--ipv6` 启用 IPv6。

```console
$ docker network create --ipv6 --ipv4=false v6net
```

在默认 bridge 网络中，无法禁用 IPv4 地址配置。

## 使用默认 bridge 网络

默认的 `bridge` 网络属于 Docker 的遗留细节，不建议在生产中使用。它的配置需要手动进行，并且存在一些[技术局限](#用户自定义-bridge-与默认-bridge-的差异)。

### 将容器连接到默认 bridge 网络

如果未通过 `--network` 指定网络，而是仅指定了网络驱动，容器将默认连接到 `bridge` 网络。连接到默认 `bridge` 网络的容器可以通信，但只能通过 IP 地址，除非使用[遗留的 `--link` 标志](../links.md)。

### 配置默认 bridge 网络

要配置默认 `bridge` 网络，请在 `daemon.json` 中指定选项。下面是一个包含若干选项的示例；仅配置你确实需要自定义的设置。

```json
{
  "bip": "192.168.1.1/24",
  "fixed-cidr": "192.168.1.0/25",
  "mtu": 1500,
  "default-gateway": "192.168.1.254",
  "dns": ["10.20.1.2","10.20.1.3"]
}
```

在该示例中：

- Bridge 的地址为 "192.168.1.1/24"（来自 `bip`）。
- Bridge 网络的子网为 "192.168.1.0/24"（来自 `bip`）。
- 容器地址将从 "192.168.1.0/25" 分配（来自 `fixed-cidr`）。


### 在默认 bridge 网络中使用 IPv6

可以在 `daemon.json` 中通过以下选项（或等效命令行）为默认 bridge 启用 IPv6：

以下三个选项仅影响默认 bridge，不用于用户自定义网络。下面的地址示例来自 IPv6 文档保留段。

- 选项 `ipv6` 必填。
- 选项 `bip6` 可选，指定默认 bridge 的地址，该地址将作为容器的默认网关，同时也指定了 bridge 网络的子网。
- 选项 `fixed-cidr-v6` 可选，指定 Docker 自动为容器分配的地址范围。
  - 前缀通常应为 `/64` 或更短。
  - 在本地网络实验时，优先使用唯一本地地址（ULA，匹配 `fd00::/8`），而非链路本地（Link Local，匹配 `fe80::/10`）。
- 选项 `default-gateway-v6` 可选；若未指定，默认使用 `fixed-cidr-v6` 子网中的第一个地址。

```json
{
  "ipv6": true,
  "bip6": "2001:db8::1111/64",
  "fixed-cidr-v6": "2001:db8::/64",
  "default-gateway-v6": "2001:db8:abcd::89"
}
```

如果未指定 `bip6`，则由 `fixed-cidr-v6` 来定义 bridge 网络的子网。若两者均未指定，将自动选择一个 ULA 前缀。

重启 Docker 以使更改生效。

## bridge 网络的连接数限制

受 Linux 内核限制，当单个网络连接的容器达到 1000 个或更多时，bridge 网络会变得不稳定，容器间通信可能中断。

更多信息参见 [moby/moby#44973](https://github.com/moby/moby/issues/44973#issuecomment-1543747718)。

## 跳过 bridge IP 地址配置

bridge 通常会被分配网络的 `--gateway` 地址，作为从该 bridge 网络到其他网络的默认路由。

通过 `com.docker.network.bridge.inhibit_ipv4` 选项，你可以创建一个不会为 bridge 分配 IPv4 网关地址的网络。这在你希望手动为 bridge 配置网关 IP（例如向 bridge 添加物理接口，并需要它拥有网关地址）时非常有用。

采用该配置后，南北向流量（进出该 bridge 网络的流量）将无法工作，除非你已经在 bridge 或其连接的设备上手动配置了网关地址。

该选项仅适用于用户自定义 bridge 网络。

## 进一步阅读

- 学习[独立网络教程](/manuals/engine/network/tutorials/standalone.md)
- 了解[从容器视角的网络](../_index.md)
- 了解 [overlay 网络](./overlay.md)
- 了解 [Macvlan 网络](./macvlan.md)
