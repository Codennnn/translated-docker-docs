title: 数据包过滤与防火墙
weight: 10
description: Docker 如何与数据包过滤、iptables 与防火墙协同工作
keywords: network, iptables, firewall
aliases:
- /network/iptables/
- /network/packet-filtering-firewalls/
---

在 Linux 上，Docker 会创建 `iptables` 与 `ip6tables` 规则，以实现网络隔离、端口发布与流量过滤。

这些规则是 Docker 桥接网络正常工作的必要条件，因此不应修改由 Docker 创建的规则。

不过，如果你的 Docker 主机暴露在互联网中，你很可能需要通过 iptables 策略来阻止未授权访问容器或主机上的其他服务。本文介绍如何实现这些策略，以及需要注意的事项。

> [!NOTE]
> 
> Docker 会为 bridge 网络创建 `iptables` 规则。
> 
> 对于 `ipvlan`、`macvlan` 或 `host` 网络，不会创建任何 `iptables` 规则。

## Docker 与 iptables 链

在 `filter` 表中，Docker 将默认策略设置为 `DROP`，并创建以下自定义 `iptables` 链：

* `DOCKER-USER`
  * 供用户自定义规则使用的占位链；该链中的规则会先于 `DOCKER-FORWARD` 与 `DOCKER` 链中的规则处理。
* `DOCKER-FORWARD`
  * Docker 网络处理的第一阶段：负责将与既有连接无关的数据包转交到其他 Docker 链，同时接受属于已建立连接的数据包。
* `DOCKER`
  * 基于正在运行容器的端口转发配置，决定不属于已建立连接的数据包是否应被接受。
* `DOCKER-ISOLATION-STAGE-1` 与 `DOCKER-ISOLATION-STAGE-2`
  * 用于在不同 Docker 网络之间实现隔离的规则。
* `DOCKER-INGRESS`
  * 与 Swarm 网络相关的规则。

在 `FORWARD` 链中，Docker 添加规则无条件跳转至 `DOCKER-USER`、`DOCKER-FORWARD` 与 `DOCKER-INGRESS` 链。

在 `nat` 表中，Docker 创建 `DOCKER` 链并添加规则，以实现地址伪装与端口映射。

### 在 Docker 规则之前添加 iptables 策略

被这些自定义链中的规则接受或拒绝的数据包，将不会再经过追加到 `FORWARD` 链的用户规则。因此，如需对这些数据包进行额外筛选，请使用 `DOCKER-USER` 链。

追加到 `FORWARD` 链的规则会在 Docker 规则之后处理。

### 匹配请求的原始 IP 与端口

当数据包到达 `DOCKER-USER` 链时，已经经过了目标网络地址转换（DNAT）处理。这意味着，此时你在 `iptables` 中使用的匹配条件只能匹配到容器的内部 IP 与端口。

如果希望依据网络请求的原始 IP 与端口进行匹配，你必须使用
[`conntrack` iptables 扩展](https://ipset.netfilter.org/iptables-extensions.man.html#lbAO)。
例如：

```console
$ sudo iptables -I DOCKER-USER -p tcp -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
$ sudo iptables -I DOCKER-USER -p tcp -m conntrack --ctorigdst 198.51.100.2 --ctorigdstport 80 -j ACCEPT
```

> [!IMPORTANT]
>
> 使用 `conntrack` 扩展可能会带来性能下降。

## 端口发布与映射

默认情况下（IPv4 与 IPv6 均如此），守护进程会阻止访问未发布的端口。已发布的容器端口会映射到主机的 IP 地址。
为实现上述行为，Docker 使用 iptables 执行网络地址转换（NAT）、端口地址转换（PAT）与地址伪装（masquerading）。

例如，`docker run -p 8080:80 [...]` 会在 Docker 主机的任意地址上的 8080 端口，与容器的 80 端口之间建立映射。容器发起的出站连接会使用 Docker 主机的 IP 地址进行伪装。

### 限制容器的外部连接

默认情况下，任何外部来源 IP 都可以连接到发布在 Docker 主机地址上的端口。

若只允许特定 IP 或网段访问容器，请在 `DOCKER-USER` 过滤链首部插入一条取反规则。例如，下面的规则会丢弃除 `192.0.2.2` 外来自所有 IP 的数据包：

```console
$ iptables -I DOCKER-USER -i ext_if ! -s 192.0.2.2 -j DROP
```

你需要将 `ext_if` 替换为主机上实际的外部网卡接口名称。也可以改为允许来自某个源子网的连接。如下规则仅允许来自子网 `192.0.2.0/24` 的访问：

```console
$ iptables -I DOCKER-USER -i ext_if ! -s 192.0.2.0/24 -j DROP
```

最后，你可以使用 `--src-range` 指定允许的 IP 地址范围（使用 `--src-range` 或 `--dst-range` 时记得同时添加 `-m iprange`）：

```console
$ iptables -I DOCKER-USER -m iprange -i ext_if ! --src-range 192.0.2.1-192.0.2.3 -j DROP
```

你可以将 `-s` 或 `--src-range` 与 `-d` 或 `--dst-range` 组合使用，同时控制源与目的。例如，如果 Docker 主机同时具有 `2001:db8:1111::2` 与 `2001:db8:2222::2`，你可以仅针对 `2001:db8:1111::2` 制定规则，并保留 `2001:db8:2222::2` 对外开放。

你可能需要允许来自许可外部地址范围之外的服务器的响应。例如，容器可能会向不被允许访问该容器服务的主机发起 DNS 或 HTTP 请求。下面的规则会接受任何属于已被其他规则接受的流的入站或出站数据包。该规则必须位于限制外部地址访问的 `DROP` 规则之前。

```console
$ iptables -I DOCKER-USER -m state --state RELATED,ESTABLISHED -j ACCEPT
```

`iptables` 配置较为复杂。更多信息请参阅 [Netfilter.org HOWTO](https://www.netfilter.org/documentation/HOWTO/NAT-HOWTO.html)。

### 直连路由（Direct routing）

端口映射可以确保已发布端口在主机的网络地址上可访问，这些地址通常对外部客户端可路由。而宿主网络通常不会为主机内部存在的容器地址设置路由。

但在某些场景（尤其是 IPv6）下，你可能希望避免使用 NAT，而是让外部网络直接路由到容器地址（“直连路由”）。

要从 Docker 主机外部访问桥接网络中的容器，首先必须通过主机上的某个地址，为该桥接网络设置路由。可通过静态路由、边界网关协议（BGP）或其他适合你网络的方式实现。例如，在本地二层网络内，远端主机可以通过 Docker 守护进程主机在本地网络中的地址，为容器网络设置静态路由。

#### 桥接网络中的容器直连路由

默认情况下，远端主机不可直接访问 Docker 的 Linux 桥接网络中的容器 IP，只能访问发布到主机 IP 地址的端口。

要在任意 Linux 桥接网络中允许任意容器的任意已发布端口被直接访问，可在 `/etc/docker/daemon.json` 中为守护进程设置 `"allow-direct-routing": true`，或使用等效的 `--allow-direct-routing` 选项。

如需允许从任意位置经直连路由访问某个特定桥接网络中的容器，请参见[网关模式](#gateway-modes)。

或者，要允许通过指定的主机网络接口对某个桥接网络进行直连路由，请在创建网络时使用以下选项：
- `com.docker.network.bridge.trusted_host_interfaces`

#### 示例

创建一个网络，使容器 IP 上的已发布端口可以从接口 `vxlan.1` 与 `eth3` 直接访问：

```console
$ docker network create --subnet 192.0.2.0/24 --ip-range 192.0.2.0/29 -o com.docker.network.bridge.trusted_host_interfaces="vxlan.1:eth3" mynet
```

在该网络中运行一个容器，并将其 80 端口发布到主机回环接口的 8080 端口：

```console
$ docker run -d --ip 192.0.2.100 -p 127.0.0.1:8080:80 nginx
```

现在，可以通过 Docker 主机的 `http://127.0.0.1:8080` 访问容器中运行在 80 端口的 Web 服务，或直接通过 `http://192.0.2.100:80` 访问。
如果连接到 `vxlan.1` 与 `eth3` 接口的远端主机，已经在 Docker 主机内部为 `192.0.2.0/24` 网络设置了路由，它们同样可以通过 `http://192.0.2.100:80` 进行访问。

#### 网关模式（Gateway modes）

bridge 网络驱动包含以下选项：
- `com.docker.network.bridge.gateway_mode_ipv6`
- `com.docker.network.bridge.gateway_mode_ipv4`

每个选项都可以设置为以下网关模式之一：
- `nat`
- `nat-unprotected`
- `routed`
- `isolated`

默认模式为 `nat`，Docker 会为每个已发布的容器端口设置 NAT 与地址伪装规则。离开主机的数据包将使用主机地址。

`routed` 模式下不会设置 NAT 或地址伪装规则，但仍会配置 `iptables`，以便只有已发布的容器端口可被访问。容器的出站数据包会使用容器自身地址，而非主机地址。

在 `nat` 模式中，当某个端口发布到指定的主机地址时，该端口只能通过拥有该地址的主机接口访问。例如，若端口发布到回环接口的地址，则远端主机无法访问。

不过在直连路由下，已发布的容器端口通常始终可被远端主机访问，除非 Docker 主机的防火墙另有限制。本地二层网络中的主机无需额外网络配置即可设置直连路由；而本地网络之外的主机，只有在网络路由器已相应配置时，才能对容器使用直连路由。

在 `nat-unprotected` 模式下，未发布的容器端口也能通过直连路由访问，不会设置端口过滤规则。该模式用于兼容旧版的默认行为。

网关模式同样会影响同一主机上、连接到不同 Docker 网络的容器之间的通信。
- 在 `nat` 与 `nat-unprotected` 模式下，其他 bridge 网络中的容器只能通过端口发布到的主机地址进行访问，不允许从其他网络进行直连路由。
- 在 `routed` 模式下，其他网络中的容器可以直接通过直连路由访问端口，无需经过主机地址。

在 `routed` 模式中，`-p` 或 `--publish` 映射中的主机端口不会被使用，主机地址仅用于决定该映射应用于 IPv4 还是 IPv6。
因此，当某条映射仅适用于 `routed` 模式时，应仅使用地址 `0.0.0.0` 或 `::`，且不应指定主机端口。如果给定了具体地址或端口，将不会影响已发布端口，并会记录一条警告日志。

`isolated` 模式仅可在网络同时使用 `--internal`（或等效选项）创建时使用。通常，`internal` 网络会为 bridge 设备分配地址，这样主机上的进程可访问该网络，网络中的容器也可访问监听在该 bridge 地址上的主机服务（包括监听在“任意”主机地址 `0.0.0.0` 或 `::` 的服务）。当使用 `isolated` 网关模式创建网络时，将不会为该 bridge 分配地址。

#### 示例

创建一个适合 IPv6 直连路由的网络，并为 IPv4 启用 NAT：
```console
$ docker network create --ipv6 --subnet 2001:db8::/64 -o com.docker.network.bridge.gateway_mode_ipv6=routed mynet
```

创建一个带已发布端口的容器：
```console
$ docker run --network=mynet -p 8080:80 myimage
```

结果如下：
- 对于 IPv4 与 IPv6，仅容器的 80 端口会开放。
- 对于 IPv6（使用 `routed` 模式），80 端口会开放在容器 IP 上；主机 IP 不会开放 8080 端口；出站数据包使用容器 IP。
- 对于 IPv4（默认 `nat` 模式），容器的 80 端口既可以通过主机 IP 的 8080 端口访问，也可以在 Docker 主机内部直接访问；但在主机外部无法直接访问容器的 80 端口。容器发起的连接会使用主机 IP 进行伪装。

In `docker inspect`, this port mapping will be shown as follows. Note that
there is no `HostPort` for IPv6, because it is using `routed` mode:
```console
$ docker container inspect <id> --format "{{json .NetworkSettings.Ports}}"
{"80/tcp":[{"HostIp":"0.0.0.0","HostPort":"8080"},{"HostIp":"::","HostPort":""}]}
```

或者，若希望该映射仅用于 IPv6（禁用对容器 80 端口的 IPv4 访问），可使用未指定的 IPv6 地址 `[::]`，并且不指定主机端口：
```console
$ docker run --network mynet -p '[::]::80'
```

### 设置容器端口的默认绑定地址

默认情况下，当容器端口映射未指定主机地址时，Docker 守护进程会将已发布的容器端口绑定到主机的所有地址（`0.0.0.0` 与 `[::]`）。

例如，下述命令会将 8080 端口发布到主机所有网络接口（IPv4 与 IPv6），从而可能对外部网络可见：

```console
docker run -p 8080:80 nginx
```

你可以更改已发布容器端口的默认绑定地址，使其默认仅对 Docker 主机可见。为此，可将守护进程配置为使用回环地址（`127.0.0.1`）。

> [!WARNING]
>
> 在 28.0.0 之前的版本中，同一二层网络（例如连接到同一交换机的主机）中的其他主机，可能访问被发布到 localhost 的端口。了解更多信息，请参见
> [moby/moby#45610](https://github.com/moby/moby/issues/45610)

若要为用户自定义 bridge 网络配置该行为，请在创建网络时使用 `com.docker.network.bridge.host_binding_ipv4` [驱动选项](./drivers/bridge.md#options)。

```console
$ docker network create mybridge \
  -o "com.docker.network.bridge.host_binding_ipv4=127.0.0.1"
```

> [!NOTE]
>
> - 将默认绑定地址设置为 `::`，表示未指定主机地址的端口绑定将适用于主机上的任意 IPv6 地址；而 `0.0.0.0` 表示任意 IPv4 或 IPv6 地址。
> - 更改默认绑定地址对 Swarm 服务无效；Swarm 服务始终暴露在 `0.0.0.0` 接口上。

#### 默认 bridge

若要为默认 bridge 网络设置默认绑定地址，请在 `daemon.json` 配置文件中设置 `"ip"` 键：

```json
{
  "ip": "127.0.0.1"
}
```

这会将默认 bridge 网络上已发布容器端口的默认绑定地址更改为 `127.0.0.1`。
重启守护进程以使更改生效。或者，也可以在启动守护进程时使用 `dockerd --ip` 标志。

## 路由器上的 Docker

在 Linux 上，Docker 需要主机启用“IP 转发”。因此，如果启动时尚未启用，Docker 会开启 `sysctl` 设置 `net.ipv4.ip_forward` 与 `net.ipv6.conf.all.forwarding`。在这样做的同时，它还会将 iptables 的 `FORWARD` 链策略设为 `DROP`。

如果 Docker 将 `FORWARD` 链默认策略设为 `DROP`，你的 Docker 主机将无法充当路由器。这是在启用 IP 转发时的推荐设置。

若要阻止 Docker 将 `FORWARD` 链策略设为 `DROP`，可在 `/etc/docker/daemon.json` 中设置 `"ip-forward-no-drop": true`，或在 `dockerd` 启动参数中添加 `--ip-forward-no-drop`。

或者，你也可以在 `DOCKER-USER` 链中添加 `ACCEPT` 规则，允许你期望转发的数据包。例如：

```console
$ iptables -I DOCKER-USER -i src_if -o dst_if -j ACCEPT
```

> [!WARNING]
>
> 在 28.0.0 之前的版本中，Docker 总是将 IPv6 的 `FORWARD` 链默认策略设置为 `DROP`。从 28.0.0 起，只有当 Docker 自身启用 IPv6 转发时，才会设置该策略。这一直是 IPv4 转发的行为。
>
> 如果在 Docker 启动前你的主机已启用 IPv6 转发，请检查主机配置以确保安全。

## 阻止 Docker 修改 iptables

你可以在[守护进程配置](https://docs.docker.com/reference/cli/dockerd/)中将 `iptables` 或 `ip6tables` 键设置为 `false`，但这并不适用于大多数用户，且很可能破坏 Docker 引擎的容器网络功能。

此时所有容器的所有端口都将直接对网络开放，而不再通过 Docker 主机 IP 地址进行映射。

完全阻止 Docker 创建 `iptables` 规则并不可行；而事后再去补齐相关规则极其复杂，超出本文范围。

## 与 firewalld 集成

如果你在启用 `iptables` 的情况下运行 Docker，并且系统已启用 [firewalld](https://firewalld.org)，Docker 会自动创建一个名为 `docker`、目标为 `ACCEPT` 的 `firewalld` 区域（zone）。

Docker 创建的所有网络接口（例如 `docker0`）都会被加入到 `docker` 区域。

Docker 还会创建一个名为 `docker-forwarding` 的转发策略，允许从任意区域（`ANY`）转发到 `docker` 区域。

## Docker 与 ufw

[Uncomplicated Firewall](https://launchpad.net/ufw)（ufw）是 Debian 与 Ubuntu 自带的防火墙前端，用于管理防火墙规则。Docker 与 ufw 使用 iptables 的方式彼此不兼容。

当你使用 Docker 发布容器端口时，往返该容器的流量会在经过 ufw 防火墙设置之前被重定向。Docker 在 `nat` 表中路由容器流量，这意味着数据包会在到达 ufw 使用的 `INPUT` 与 `OUTPUT` 链之前就被转发。由于数据包在防火墙规则生效前已被路由，从效果上看你的防火墙配置会被忽略。
